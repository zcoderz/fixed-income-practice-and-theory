# Chapter 44: CDS Relative Value Trading Frameworks

---

## Introduction

A corporate's 3-year CDS trades at 80bp, its 5-year at 120bp. Is that steep or flat? A hedge fund manager sees the cash bond trading 30bp wider than the CDS. Is that a buying opportunity or a warning sign? A credit trader notices that senior and subordinated CDS spreads have compressed. Should she put on a capital structure trade? The equity desk sees implied volatility on puts at 60% while the credit desk sees 5Y CDS at 150bp—is that relationship in line, or is one market mispricing the credit risk?

These questions sit at the heart of CDS relative value trading. Unlike directional credit positions that simply bet on spreads widening or tightening, relative value (RV) trades attempt to exploit mispricings between related instruments while hedging out broader market exposure. The appeal is obvious: if you can identify a temporary dislocation between the 3-year and 5-year CDS curve, why take outright default risk when you can trade the relationship instead?

The danger lies in what appears market-neutral but is not. O'Kane emphasizes in his treatment of CDS risk management that a CDS position carries multiple, distinct risks: "sensitivities to changes in the credit spread curve, changes in the Libor rate curve, changes in the recovery rate, the passage of time, and the risk of a default event." A curve trade that neutralizes total CS01 may still carry massive jump-to-default risk. A basis trade that looks like pure arbitrage may blow up when funding markets seize. Stress episodes (e.g., the global financial crisis) are reminders that "market-neutral" can turn into correlation + liquidity + funding risk very quickly.

This chapter takes a risk-first approach to CDS relative value. Building on the CDS mechanics from Chapter 38, the pricing framework from Chapter 41, and the risk measures from Chapter 43, we develop a systematic framework for translating any RV idea into explicit risk exposures, designing hedges that target specific risks, identifying failure modes before they materialize, and constructing verification tests that expose hidden vulnerabilities. The framework applies equally to curve trades (steepeners and flatteners), cash-CDS basis trades, capital structure trades (senior versus subordinated), equity-credit arbitrage (Merton-based trades), and index versus single-name relative value. For index-specific mechanics, we connect to Chapter 45's treatment of CDS indices.

We begin with the core risk concepts that underpin all CDS positions: CS01, jump-to-default, recovery sensitivity, and theta. We then apply these concepts systematically to the major RV trade types, working through the mechanics of constructing and hedging each position. We conclude with equity-credit relative value—a domain that bridges the credit derivatives and equity derivatives desks through the Merton framework—and historical case studies that illuminate how seemingly "hedged" positions can fail catastrophically.

---

## 44.1 Core Risk Concepts for CDS Relative Value

Before constructing any relative value trade, we must establish a precise vocabulary for the risks we will be taking and hedging. O'Kane's Chapter 8 provides the canonical framework for CDS risk management, which we summarize here with emphasis on RV applications.

### 44.1.1 The CDS Mark-to-Market Identity

The mark-to-market value of a CDS position depends on the difference between the contractual spread and the current market spread. O'Kane provides the fundamental relationship:

$$\boxed{V(t) \approx (S_{\text{market}}(t,T) - S_{\text{contract}}) \cdot \text{RPV01}(t,T) \cdot N}$$

where $S_{\text{market}}(t,T)$ is the current par spread for maturity $T$, $S_{\text{contract}}$ is the contractual coupon, $\text{RPV01}(t,T)$ is the risky PV01 (present value of 1bp of spread payments), and $N$ is the notional. This identity is central to RV analysis: it shows that spread differences translate into P&L through the RPV01 multiplier.

The RPV01 itself is a survival-weighted annuity. Using O'Kane's discrete approximation (Ch 6.4):

$$\text{RPV01}(t,T) = \frac{1}{2} \sum_{n=1}^{N} \Delta(t_{n-1}, t_n) \cdot Z(t, t_n) \cdot (Q(t, t_{n-1}) + Q(t, t_n))$$

where $\Delta$ is the accrual factor, $Z$ is the discount factor, and $Q$ is the survival probability. The trapezoidal weighting of survival probabilities accounts for premium accrued at default.

### 44.1.2 Credit DV01 (CS01)

The credit DV01, often called CS01, measures the sensitivity of the CDS value to parallel shifts in the spread curve. O'Kane (Ch 8.3.2) defines it with a sign convention:

$$\text{Credit DV01} = -(V(S + 1\text{bp}) - V(S))$$

The negative sign ensures that a short protection position (which loses value when spreads widen) has positive Credit DV01. For a long protection position at par:

$$\text{CS01} \approx N \cdot 0.0001 \cdot \text{RPV01}(t,T)$$

**Critical for RV:** CS01 is a scalar that summarizes sensitivity to parallel shifts. But RV trades often involve curve positions where the front and back ends move differently. For these, we need bucket or key-rate CS01s, analogous to key-rate DV01s in the rates world (covered in Chapter 14).

O'Kane provides a summary of CDS risk sensitivities in Table 8.3 of his book, which we adapt here:

**Table 44.1: CDS Risk Summary (adapted from O'Kane Table 8.3)**

| Risk Measure | Definition | Sign Convention |
|--------------|------------|-----------------|
| Credit DV01 | Sensitivity to 1bp parallel spread shift | Positive for short protection |
| Interest Rate DV01 | Sensitivity to 1bp rate shift | Small; usually second-order |
| Recovery DV01 | Sensitivity to 1% recovery change | Depends on position direction |
| Theta | Sensitivity to time passage | Positive for short protection |
| VOD | Value change on immediate default | Gain for long protection |

### 44.1.3 Bucket CS01 (Credit Key-Rate Style)

Just as key-rate DV01s decompose interest rate sensitivity into maturity buckets, bucket CS01s decompose credit spread sensitivity. For a CDS position, we can split the RPV01 by time intervals:

| Bucket | RPV01 Contribution | Bucket CS01 |
|--------|-------------------|-------------|
| 0-2 years | $\text{RPV01}_{0-2}$ | $N \cdot 0.0001 \cdot \text{RPV01}_{0-2}$ |
| 2-5 years | $\text{RPV01}_{2-5}$ | $N \cdot 0.0001 \cdot \text{RPV01}_{2-5}$ |
| 5-10 years | $\text{RPV01}_{5-10}$ | $N \cdot 0.0001 \cdot \text{RPV01}_{5-10}$ |

**Why this matters:** A curve steepener that is CS01-neutral in aggregate may have large opposite-sign bucket exposures. This is the intended bet. But failing to report bucket exposures masks the true risk profile.

### 44.1.4 Jump-to-Default (JTD) and Value on Default (VOD)

The VOD measures the change in position value upon immediate default. O'Kane (Ch 8.3.7) defines it as:

$$\boxed{\text{VOD} = \begin{cases} -V(t) - (1-R) + \Delta_0 S_0 & \text{for protection seller} \\ -V(t) + (1-R) - \Delta_0 S_0 & \text{for protection buyer} \end{cases}}$$

where $\Delta_0 S_0$ is the accrued premium at default. If the position is at par ($V(t) \approx 0$) and we ignore accrued premium:

$$\text{JTD}_{\text{buy protection}} \approx +N(1-R)$$
$$\text{JTD}_{\text{sell protection}} \approx -N(1-R)$$

O'Kane emphasizes that "if default follows a gradual but significant widening in credit spreads, then we should find that the VOD is small." The VOD is therefore "a measure of the 'unexpected shock' of the default." This insight is crucial: a name that gradually widens from 100bp to 2000bp before defaulting will have a small VOD because the MTM already reflects the elevated default probability.

**Critical for RV:** A curve trade with zero net CS01 can still have massive net JTD if the notionals are unbalanced. This is the single most important risk often overlooked in curve trades.

### 44.1.5 Recovery Sensitivity

The protection payoff depends on $(1-R)$. O'Kane notes that practitioners "calculate the change in the mark-to-market value for an absolute change of 1% in the recovery rate. This is known as the recovery rate DV01."

Recovery sensitivity affects both:
1. **The default payoff:** Higher recovery means smaller protection payment
2. **The calibrated hazard rate:** For fixed spreads, $h \approx S/(1-R)$ via the credit triangle (Chapter 43), so recovery changes affect the implied default probability

O'Kane presents empirical recovery data from Altman et al. (Table 3.2 in his book), which provides essential context for capital structure trades:

**Table 44.2: Recovery Rates by Seniority (O'Kane Table 3.2, Altman et al. data)**

| Seniority | Mean Recovery | Median Recovery | Std Dev |
|-----------|--------------|-----------------|---------|
| Senior Secured | 51.08% | 51.63% | 25.18% |
| Senior Unsecured | 34.89% | 42.27% | 26.08% |
| Senior Subordinated | 30.17% | 32.35% | 24.05% |
| Subordinated | 29.03% | 31.96% | 20.29% |
| Junior Subordinated | 18.19% | 14.29% | 14.16% |

O'Kane notes that "the standard deviation of the recovery rate distributions is rather broad," implying that using point estimates for recovery introduces significant model risk.

### 44.1.6 Theta (Time Decay)

Theta measures the sensitivity to the passage of time with curves unchanged:

$$\Theta = \frac{\partial V}{\partial t}$$

For a short protection position (receiving premium), theta is generally positive: the position gains value as time passes without default. For a long protection position, theta is negative.

O'Kane (Ch 8.3.5) provides insight into theta dynamics. He notes that for a protection seller, theta "is made up of three components: the value of the premium receipt, the change in the protection leg value, and the change in the premium leg value." The interplay of these components means theta is not constant over time. O'Kane's Figure 8.1 shows that theta for a protection seller position tends to be highest (most positive) early in the contract's life and diminishes as the contract approaches maturity.

**For RV:** Carry and rolldown decompositions use theta logic. "Carry" refers to net premium accrual; "rolldown" refers to the MTM change from moving along a static curve as maturity shortens. These decompositions are conditional on no curve movement and no default—they are accounting tools, not return forecasts.

---

## 44.2 The CDS-Cash Basis: Drivers and Dynamics

The CDS-cash basis is one of the most actively traded RV relationships. O'Kane (Ch 5.6) defines it as:

$$\text{CDS basis} = S_{\text{CDS}} - S_{\text{Bond}}$$

where the bond spread measure requires specification (Z-spread, asset swap spread, or par floater spread). O'Kane notes that "there are now a significant number of market participants who actively trade the default swap basis, viewing it as a new relative value opportunity."

### 44.2.1 Fundamental Drivers of the Basis

O'Kane (Ch 5.6.1) distinguishes between **fundamental factors** (contractual differences) and **market factors** (liquidity and technicals). The fundamental factors are:

**1. Funding:** "Default swaps are unfunded transactions and bonds are funded. For the same spread, CDS are favoured by investors who have funding costs above Libor while bonds are favoured by those who fund below. As a result, the basis is affected by the differing funding levels of the market participants."

**2. Delivery Option:** "For a protection buyer who is using the CDS to hedge a specific security, the value of the delivery option following a credit event is simply the difference between the value of the security they hold and the value of the cheapest deliverable." O'Kane provides an example: if a protection buyer holds an asset trading at \$43 and can deliver a different asset trading at \$37, they can profit by \$6 from the delivery option.

**3. Technical Default:** "The standard credit events may be viewed as being broader than those which constitute default on a bond. As a result, protection sellers in a CDS may demand a higher spread."

**4. Loss on Default:** "The protection payment on a CDS following a credit event is a fraction $(1-R)$ of the face value. The default risk on a bond purchased at a full price of $P$ is a loss of $P-R$."

**5. Premium Accrued at Default:** "Following a credit event, a CDS pays the protection seller the premium which has accrued since the previous payment date. However, when a bond defaults, the owner's claim is on the face value, and so any accrued coupons are lost."

### 44.2.2 Market Factors Driving the Basis

The market factors O'Kane identifies (Ch 5.6.2) are equally important for understanding basis dynamics:

**1. Relative Liquidity:** "The liquidity of the cash and CDS markets is different across the term structure... The CDS market has its liquidity points at fixed term points with most liquidity concentrated at the three-, five-, seven- and 10-year maturity 'IMM' dates."

**2. Synthetic CDO Technical:** "When dealers issue synthetic CDOs, they then hedge their spread risk by selling CDS credit protection on each of the 100 or more credits in the reference portfolio. CDO issuance is therefore usually accompanied by a tightening (reduction) in CDS spreads. This reduces the CDS basis."

**3. Demand for Protection:** "It is much easier to go short a credit by buying protection in the CDS market than by shorting a cash bond. As a result, if there is negative news on a credit, it will tend to result in a flurry of protection buying in the CDS market thus increasing the CDS basis."

**4. Funding Risk:** "Since the CDS is unfunded, it effectively locks in a funding rate of Libor flat until maturity or a credit event, whichever occurs first. There is therefore no funding risk."

**5. Convertible Bond Arbitrage:** O'Kane notes that convertible arbitrage strategies can affect the basis: hedge funds buying convertible bonds and hedging credit risk via CDS create basis effects.

**6. Repo Market Conditions:** The availability and cost of financing in the repo market affects the relative attractiveness of cash bonds versus synthetic exposure.

O'Kane concludes: "All of these fundamental and market factors are reasons why the CDS and cash market spreads should diverge. It is difficult to assign degrees of importance to the individual factors as it is hard to tease them apart empirically."

> **Deep Dive: The Negative Basis Trade ("The Package")**
>
> This is one of the most famous RV trades.
>
> *   **The Setup** (illustrative numbers):
>     *   Buy the Bond (yields roughly L + 150bp).
>     *   Buy CDS Protection (pay roughly 120bp).
>     *   **Net**: you earn roughly L + 30bp *risk-free* (theoretically).
> *   **Why does it exist?**: Usually because "cash is king." In a crisis, people sell bonds to raise cash, driving bond yields up. They don't sell CDS because it generates no cash.
> *   **The Catch**: You need to fund the bond purchase (Repo). If Repo rates spike, your 30bp profit vanishes.

> **Analogy: The Lock-In**
>
> The Negative Basis trade is like finding a \$100 bill on the floor of a room, but the door locks behind you.
>
> *   **The \$100**: The 30bp arbitrage profit.
> *   **The Lock**: You own an illiquid bond. You cannot exit easily without paying a liquidity premium.
> *   **The Risk**: If the building catches fire (funding crisis), you are trapped inside with your \$100 bill.

### 44.2.3 Why Basis Trades Can Fail

The basis trade appears to offer near-arbitrage: buy a cheap bond, hedge with CDS, collect the positive carry from the basis. The risks are:

1. **Funding blowout:** If repo markets seize, the funding leg of the trade can become prohibitively expensive or impossible to roll.

2. **Correlation breakdown:** The bond and CDS may not move together. A liquidity shock can widen bond spreads while CDS spreads remain stable, or vice versa.

3. **JTD mismatch:** If the CDS notional is sized for CS01 neutrality rather than JTD neutrality, default produces a P&L discontinuity.

4. **Delivery option uncertainty:** At default, the cheapest deliverable may trade far from expectations, affecting realized recovery.

5. **Counterparty risk:** The CDS hedge is only as good as the counterparty providing it. In systemic stress, counterparty credit becomes correlated with the underlying credit.

### 44.2.4 Funding Cost Deep Dive

The economics of a basis trade depend critically on funding levels. Consider how changes in repo rates affect trade viability:

In a simple carry decomposition (ignoring haircuts and mark-to-market effects), the net carry is approximately:

$$\\text{Net carry (bp)} \\approx \\text{Bond spread} - \\text{Repo funding cost} - \\text{CDS premium}$$

**Table 44.3: Funding Level vs Net Carry (Illustrative)**

| Repo Funding Cost | Bond Spread (L+X) | CDS Premium | Net Carry (bp) |
|-------------------|-------------------|-------------|----------------|
| L + 10bp | 150bp | 120bp | +20bp positive |
| L + 25bp | 150bp | 120bp | +5bp positive |
| L + 50bp | 150bp | 120bp | -20bp (breakeven) |
| L + 100bp | 150bp | 120bp | -70bp (losing) |
| L + 150bp | 150bp | 120bp | -120bp (severely losing) |

The table reveals the funding cliff: a trade that earns carry at normal funding levels can become deeply negative as funding costs rise. In a funding stress, repo levels and haircuts can move sharply and financing can become unavailable precisely when you want to hold the hedge.

> **Practitioner Note: The Funding Trap**
>
> Middle-office professionals often see basis trades marked at theoretical value without explicit funding cost attribution. When funding costs spike, the "unexplained P&L" is exactly the funding component that wasn't being tracked separately. The lesson: always decompose basis trade P&L into (1) basis convergence, (2) carry, and (3) funding cost—never lump them together.

---

## 44.3 Credit Curve Trades: Steepeners and Flatteners

Curve trades express views on the shape of the credit term structure. They are constructed by taking opposite positions at different maturities.

### 44.3.1 Defining Curve Positions

For maturities $T_1 < T_2$:

| Trade | Definition | Instrument Position |
|-------|------------|---------------------|
| **Steepener** | Profits if $S(T_2) - S(T_1)$ increases | Buy protection at $T_2$, sell protection at $T_1$ |
| **Flattener** | Profits if $S(T_2) - S(T_1)$ decreases | Sell protection at $T_2$, buy protection at $T_1$ |

The steepener profits when the curve gets steeper (long end widens relative to short end). The flattener profits when the curve flattens.

### 44.3.2 CS01-Neutral Construction

To hedge against parallel curve moves, we choose notionals such that net CS01 equals zero:

$$N_1 \cdot \text{CS01}^{(1)} + N_2 \cdot \text{CS01}^{(2)} = 0$$

where $\text{CS01}^{(1)}$ and $\text{CS01}^{(2)}$ have opposite signs (one buy, one sell protection). Solving:

$$\boxed{\frac{N_1}{N_2} = -\frac{\text{CS01}^{(2)}}{\text{CS01}^{(1)}} = -\frac{\text{RPV01}(T_2)}{\text{RPV01}(T_1)}}$$

Since $\text{RPV01}(T_2) > \text{RPV01}(T_1)$ for $T_2 > T_1$, we need more notional at the short maturity.

### 44.3.3 The JTD Problem in Curve Trades

CS01 neutrality does not imply JTD neutrality. Consider a 5Y vs 1Y steepener with:
- Buy 5Y protection: $N_5 = 10\text{mm}$, $\text{RPV01}(5) = 4.7$
- Sell 1Y protection: notional chosen for CS01 neutrality

The CS01-neutral notional ratio is approximately $4.7 / 1.0 = 4.7$, so $N_1 \approx 47\text{mm}$.

**JTD calculation:**
- 5Y buy protection JTD: $+10 \times 0.60 = +6.0\text{mm}$ (gain at default)
- 1Y sell protection JTD: $-47 \times 0.60 = -28.2\text{mm}$ (loss at default)
- **Net JTD: $-22.2\text{mm}$**

This trade is massively short default. If the reference entity defaults, the loss overwhelms any curve P&L. This is the central warning: **curve trades can be dominated by default-event risk even when CS01-neutral**.

### 44.3.4 When Curve Trades Historically Failed

Curve trades have historically failed in several scenarios:

1. **Sudden default:** A company defaults before expected. The CS01-neutral steepener with large net short JTD produces catastrophic losses.

2. **Curve inversion:** Credit curves can invert (front end wider than back) during acute stress, as the market prices high near-term default probability.

3. **Liquidity divergence:** The 5Y point typically has better liquidity than off-the-run maturities. In stress, liquidity can evaporate at the short end while remaining at 5Y, causing basis moves that overwhelm the curve bet.

4. **Roll and technical effects:** CDS indices roll semi-annually. Around roll dates, the market's preferred maturity points shift, creating technical moves that are unrelated to credit fundamentals.

### 44.3.5 Carry and Rolldown for Curve Trades

Under a "curves unchanged" assumption, we can decompose expected P&L:

**Carry:** Net premium accrual from premium receipts minus payments. For a steepener (buy long end, sell short end), you typically pay premium on the long leg and receive on the short leg. Whether net carry is positive depends on the spread levels and notional ratios.

**Rolldown:** The MTM change from the remaining maturities shortening along a static curve. If the curve is upward sloping, buying protection at 5Y and letting it roll to 4.9Y produces a gain (you're now at a lower spread point on the curve).

**Caution:** These decompositions are conditional on no curve movement and no default. They are accounting tools, not return forecasts.

---

## 44.4 Capital Structure Trades: Senior vs Subordinated

Capital structure trades exploit the relationship between CDS referencing different seniority tiers of the same issuer.

### 44.4.1 The Spread Ratio Approximation

For senior and subordinated CDS on the same entity, O'Kane provides a rough relationship:

$$\frac{S_{\text{sub}}}{S_{\text{senior}}} \approx \frac{1 - R_{\text{sub}}}{1 - R_{\text{senior}}}$$

and explicitly notes this is "very rough in practice."

Using the recovery data from Table 44.2:
- Senior unsecured: mean recovery 34.89%
- Subordinated: mean recovery 29.03%

The theoretical ratio would be: $(1-0.29)/(1-0.35) = 0.71/0.65 = 1.09$. But empirically, subordinated spreads are often 1.5x to 3x senior spreads, implying the market prices additional factors beyond the simple recovery differential.

### 44.4.2 Why the Relationship is "Very Rough"

Several factors cause deviations from the theoretical spread ratio:

1. **Liquidity differences:** Senior CDS is far more liquid than subordinated CDS for most issuers.

2. **Recovery uncertainty:** Actual recoveries vary widely around averages. O'Kane notes "the standard deviation of the recovery rate distributions is rather broad" and "the absolute priority rule (APR) is not always obeyed in the US, meaning that in certain circumstances, holders of subordinated debt may recover more than holders of more senior debt."

3. **Technical supply/demand:** Bank regulatory capital rules, CDO structuring, and index composition create supply/demand imbalances that vary by seniority.

4. **Different default triggers:** Some credit events may trigger protection on one seniority tier but not another, particularly around restructuring.

### 44.4.3 Constructing a Capital Structure Trade

A typical trade expresses a view on the senior-sub spread differential:

- **Compression trade:** Buy senior protection, sell subordinated protection. Profits if the sub/senior spread ratio narrows.
- **Decompression trade:** Sell senior protection, buy subordinated protection. Profits if the sub/senior spread ratio widens.

**Sizing:** CS01-neutral sizing uses the same hedge ratio logic:

$$\frac{N_{\text{senior}}}{N_{\text{sub}}} = -\frac{\text{CS01}_{\text{sub}}}{\text{CS01}_{\text{senior}}}$$

**JTD consideration:** Because recovery assumptions differ, JTD depends on both notional and assumed recovery:
- Sub buy protection JTD: $+N_{\text{sub}}(1-R_{\text{sub}})$
- Senior sell protection JTD: $-N_{\text{senior}}(1-R_{\text{senior}})$

Even with CS01 neutrality, the different recoveries create JTD imbalance.

### 44.4.4 When Capital Structure Trades Widen or Tighten

Events that affect the senior-sub spread:

**Compression (sub/senior ratio tightens):**
- Credit improvement: as default risk falls, the absolute spreads compress and the recovery differential matters less
- Regulatory changes favoring sub debt

**Decompression (sub/senior ratio widens):**
- Credit deterioration: higher default probability amplifies the recovery differential
- Distress: market prices in that sub will recover less than expected
- Liquidity crisis: sub CDS liquidity evaporates first

### 44.4.5 Event Risk: LBO and M&A Effects

> **Practitioner Note: The LBO Trade**
>
> Leveraged buyout (LBO) announcements create distinctive patterns in credit spreads that sophisticated traders anticipate. When a company is taken private via LBO, the new owners typically add substantial debt to the capital structure, which affects existing creditors asymmetrically.
>
> **The Mechanics:**
> - New secured debt is layered on top of existing obligations
> - Existing senior unsecured debt becomes effectively subordinated
> - Cash flow available for debt service is stretched across more claims
> - Covenants may be weakened or removed
>
> **Spread Reactions (stylized):**
> - Senior spreads can widen materially on an LBO announcement (magnitude depends on leverage, structure, and market regime)
> - Subordinated spreads often widen more, so the sub/senior ratio can increase (decompression)
>
> **The "LBO Trade":**
> Before LBO announcements, event-driven funds identify likely targets based on:
> - Strong free cash flow (can service new debt)
> - Low existing leverage (room for more debt)
> - Stable, predictable business (attractive to PE sponsors)
> - Cheap valuation (attractive entry point)
>
> They then buy protection on senior CDS and/or put on curve steepeners, anticipating the spread widening and decompression that accompany leveraging events.
>
> **Curve Implications:**
> LBO risk manifests in the 5Y point but not the 1Y point—deals take time to consummate. This creates curve steepening opportunities: buy 5Y protection, sell 1Y protection, anticipating that announcement effects hit the long end first.
>
> **Warning:** Event risk is distinct from credit deterioration. A company can be a high-quality credit yet have significant LBO risk precisely because its quality makes it an attractive PE target.

---

## 44.5 Equity-Credit Relative Value: The Merton Framework

One of the most sophisticated RV strategies bridges the credit and equity derivatives markets through the Merton model of default. This section develops the theory and practice of equity-credit arbitrage.

### 44.5.1 The Merton Model: Equity as a Call Option

The foundational insight comes from Merton's 1974 paper, which Hull summarizes clearly: "A company's equity is an option on the assets of the company."

Consider a firm with:
- $V_0$: Current value of company's assets
- $V_T$: Value of assets at time $T$
- $E_0$: Current value of equity
- $D$: Amount of debt due at time $T$
- $\sigma_V$: Volatility of assets
- $\sigma_E$: Volatility of equity

If $V_T < D$ at maturity, the company defaults—equity holders receive nothing. If $V_T > D$, equity holders receive $V_T - D$ after paying off the debt. This means:

$$E_T = \max(V_T - D, 0)$$

This is exactly the payoff of a call option on firm assets with strike price equal to the debt. Hull provides the pricing equation:

$$\boxed{E_0 = V_0 N(d_1) - D e^{-rT} N(d_2)}$$

where:
$$d_1 = \frac{\ln(V_0/D) + (r + \sigma_V^2/2)T}{\sigma_V \sqrt{T}}$$
$$d_2 = d_1 - \sigma_V \sqrt{T}$$

and $N(\cdot)$ is the cumulative normal distribution function.

### 44.5.2 The Equity Volatility Link

The equity and asset volatility are linked through the option's delta. Hull derives:

$$\boxed{\sigma_E E_0 = N(d_1) \sigma_V V_0}$$

This equation, combined with the equity pricing formula, gives us two equations in two unknowns ($V_0$ and $\sigma_V$). Given observable equity value $E_0$ and equity volatility $\sigma_E$, we can solve for the unobservable firm value and asset volatility.

Hull provides a worked example: "The value of a company's equity is \$3 million and the volatility of the equity is 80%. The debt that will have to be paid in one year is \$10 million. The risk-free rate is 5% per annum." Solving the simultaneous equations yields $V_0 = \$12.40$ million and $\sigma_V = 21.23\%$. The implied probability of default is $N(-d_2) = 12.7\%$.

### 44.5.3 Distance to Default

Hull introduces the key practitioner concept: "The term distance to default has been coined to describe the output from Merton's model. This is the number of standard deviations the asset price must change for default to be triggered."

$$\boxed{\text{Distance to Default} = d_2 = \frac{\ln(V_0/D) + (r - \sigma_V^2/2)T}{\sigma_V \sqrt{T}}}$$

The distance to default can be interpreted as follows: if the firm's assets are currently 2.5 standard deviations above the default barrier, the DD is 2.5. As the company becomes more leveraged or volatile, DD declines and default probability rises.

Hull notes that "Moody's KMV and Kamakura provide a service that transforms a default probability produced by Merton's model into a real-world default probability." The raw Merton probabilities must be calibrated to empirical default frequencies to be useful for prediction.

### 44.5.4 Credit Spreads from Merton

The Merton model directly implies credit spreads. From Hull: the credit spread on a $T$-year zero-coupon bond is:

$$\text{Credit Spread} = -\frac{1}{T}\ln\left[N(d_2) + \frac{N(-d_1)}{L}\right]$$

where $L = D e^{-rT}/V_0$ is the leverage ratio in present value terms.

This provides the crucial link: equity volatility $\sigma_E$ → asset volatility $\sigma_V$ → distance to default $d_2$ → credit spread. If the equity market is pricing one level of volatility and the CDS market is pricing a different credit spread, an arbitrage opportunity exists.

### 44.5.5 Capital Structure Arbitrage: The Trade

The Volatility Surface book describes the arbitrage mechanism: "Capital structure arbitrage is the term used to describe the fashion for arbitraging equity claims against fixed income and convertible claims. At its simplest, the trader looks to see if equity puts are cheaper than credit derivatives and if so buys the one and sells the other."

**The Setup:**

Under the Merton framework, a put option on equity has value related to the firm's default probability. A CDS provides protection against default. In principle, they should be priced consistently.

**Key insight from put-call parity under default risk:**

The Volatility Surface book derives:

$$P_0 = P_I + K(B_0 - B_I)$$

where $P_0$ is the value of a risk-free put, $P_I$ is the value of an issuer-written put, $B_0$ is a risk-free bond, and $B_I$ is the issuer's risky bond. The difference $K(B_0 - B_I)$ is approximately the value of default protection—i.e., a CDS.

**The Arbitrage:**

The Volatility Surface explains: "Taking advantage of the market maker's lack of understanding, the trader buys an equity option on the exchange at a 'very high' (but, of course, insufficiently high) implied volatility and sells a default put on the same stock in the credit derivatives market locking in a risk-free return."

This actually occurred: "hedge funds were able to lock in risk-free gains for a period of time. During this period, market makers saw what were to them extremely steep volatility skews get even steeper and they lost money."

### 44.5.6 The CreditGrades Model

The basic Merton model has a problem: it generates almost no short-dated credit spreads. The firm value must diffuse to the default barrier, which takes time. For short maturities, this probability is negligible.

The CreditGrades model (described in the Volatility Surface book) resolves this by making the default barrier uncertain. The model assumes:

- Firm value $V$ follows geometric Brownian motion: $dV_t/V_t = \sigma dW$
- The default barrier is $LD$ where $D$ is debt per share and $L$ is recovery rate
- Critically, $L$ is lognormally distributed with mean $\bar{L}$ and standard deviation $\lambda$

This uncertainty in the recovery/barrier level generates meaningful short-dated credit spreads. The survival probability under CreditGrades is:

$$P_t = N\left(-\frac{A_t}{2} + \frac{\log d}{A_t}\right) - d \cdot N\left(-\frac{A_t}{2} - \frac{\log d}{A_t}\right)$$

where:
$$d = \frac{V_0 e^{\lambda^2}}{\bar{L}D}, \quad A_t^2 = \sigma^2 t + \lambda^2$$

The CreditGrades model relates equity volatility to credit spreads in a more realistic way than basic Merton, enabling practitioners to identify when equity vol looks "cheap" or "rich" relative to credit.

### 44.5.7 Example: Goodyear Tire (GT)

The Volatility Surface provides a detailed example fitting the Merton model to Goodyear Tire options (October 2004). With GT's stock at \$9.40 and high credit spreads (5Y CDS over 5%), the book fits Merton parameters:

$$\lambda = 0.01934, \quad \sigma = 39.46\%$$

The fitted implied volatilities match the market well for low strikes:

| Strike | Market Vol | Merton Vol |
|--------|-----------|------------|
| 2.50 | 147% | 145% |
| 5.00 | 81% | 86% |
| 7.50 | 53% | 51% |
| 10.00 | 42% | 43% |

The book notes: "most of the volatility skew for stocks with high credit spreads can be ascribed to default risk." However, Merton generates no right-wing structure (high strikes)—it produces a pure steep downside skew and flat upside, whereas empirical surfaces show some upside structure from stochastic volatility.

### 44.5.8 When Equity-Credit Arbitrage Fails

> **Practitioner Note: The Capital Structure Arbitrage Blow-Up**
>
> Capital structure arbitrage was highly profitable from 2003-2006 as hedge funds exploited systematic mispricings between equity options and CDS. However, the strategy faced serious challenges:
>
> **What Went Wrong:**
> 1. **Model risk:** Merton is a simplification. Real defaults involve complex legal processes, not clean option exercise.
> 2. **Correlation regime change:** The correlation between equity and credit can shift dramatically in crisis. Equity may rally on M&A rumors while credit widens on leverage concerns.
> 3. **Liquidity mismatch:** Equity options are exchange-traded with tight spreads; CDS is OTC with wider spreads and counterparty risk.
> 4. **Jump risk:** Merton assumes diffusive asset value process. In reality, firms can experience sudden jumps (fraud revelations, regulatory actions) that violate the model.
> 5. **Convergence timing:** The arbitrage may require long holding periods for convergence, during which funding costs accumulate.
>
> **The GM/Ford Correlation Trade (2005):** Many funds had sold protection on auto credits while buying equity puts, betting on credit-equity consistency. When GM and Ford were downgraded to junk in May 2005, credit spreads exploded but equity did not fall proportionately—the hedge didn't work as expected. Funds positioned for the "normal" relationship suffered significant losses when correlation broke down.

---

## 44.6 Index vs Single-Name Relative Value

The relationship between a CDS index and its constituent single-name CDS creates another RV opportunity. This section provides a brief treatment; Chapter 45 covers index mechanics in depth.

### 44.6.1 Intrinsic Spread and Index Basis

O'Kane (Ch 10.5) defines the intrinsic value of a CDS index as the sum of the values of its constituent CDS. The intrinsic spread is the spread that makes the intrinsic value equal to zero.

The **index basis** is:

$$\text{Index basis} = S_{\text{index}} - S_{\text{intrinsic}}$$

O'Kane shows that the intrinsic spread can be calculated as:

$$\boxed{S_{\text{intrinsic}} = \frac{\sum_{m=1}^{M} S_m \cdot \text{RPV01}_m}{\sum_{m=1}^{M} \text{RPV01}_m}}$$

where the weighting by RPV01 accounts for the fact that higher-spread names contribute more to index value. This is an RPV01-weighted average, not a simple average of spreads.

**Table 44.4: Intrinsic vs Average Spread**

| Spread Type | Formula | When Different |
|-------------|---------|----------------|
| Simple Average | $\bar{S} = \frac{1}{M}\sum_{m=1}^{M} S_m$ | All names equal weight |
| Intrinsic | $S_{\text{intrinsic}} = \frac{\sum S_m \cdot \text{RPV01}_m}{\sum \text{RPV01}_m}$ | Higher-spread names get more weight |

The difference between the intrinsic spread and the simple average can be material when spread dispersion is high.

### 44.6.2 Drivers of Index Basis

The index basis can be positive or negative depending on:

1. **Liquidity premium:** The index is more liquid than individual constituents, so it may trade tighter.

2. **Transaction costs:** Replicating the index via singles incurs more transaction costs.

3. **Supply/demand:** Macro hedgers often use indices, creating technical demand.

4. **CDO issuance:** Synthetic CDO issuance creates supply of index protection.

5. **Skew effects:** When a few names are much wider than the rest, the intrinsic spread is pulled higher, potentially creating a negative basis.

### 44.6.3 Trading the Index Basis

The trade involves going long (short) the index versus short (long) the constituents:

- **Long basis:** Buy index protection, sell protection on constituents
- **Short basis:** Sell index protection, buy protection on constituents

Sizing requires matching the overall CS01, typically by scaling constituent positions to match the index notional divided by the number of names.

---

## 44.7 Historical Case Studies

Understanding past failures is essential for risk management. These case studies illustrate how "hedged" positions can produce catastrophic losses.

### 44.7.1 Case Study: Funding-Liquidity Unwind in a Stress Episode

> **Practitioner Note: The Basis Trade Failure Mode**
>
> **The Setup (stylized):** Negative basis trades can look like "money machines." You buy a bond that screens cheap versus CDS, buy CDS protection, and expect to earn carry plus basis convergence.
>
> **What goes wrong in stress:**
> - **Financing deteriorates:** repo rates and/or haircuts rise and funding may not roll.
> - **Liquidity vanishes in cash bonds:** selling the bond can become costly; marks gap out; bid-ask widens.
> - **Margin and risk limits shorten the holding period:** interim losses trigger margin calls and position cuts.
> - **Hedge effectiveness is not enough:** even if CDS hedges default risk, P&L can be dominated by funding and liquidity.
>
> **Lesson:** Basis trades are not just credit trades. They are funding/liquidity trades disguised as arbitrage. Always decompose P&L into (1) basis/spread, (2) carry/theta, and (3) funding.
>
> **What risk reports often show vs reality:**
> - Reports: CS01 neutral, small VaR, positive carry
> - Reality: exposure to haircuts, funding roll risk, and liquidity spirals

### 44.7.2 Case Study: Equity–Credit Disconnect in Bankruptcy (Illustrative)

> **Practitioner Note: Timing Risk in Equity–Credit RV**
>
> It can happen that a company's credit trades at distressed levels while its equity price rallies sharply. At first glance this looks like a free arbitrage ("equity must be worthless if the debt is impaired"), but in practice the trade can be dangerous.
>
> **Why equity can rally while credit stays distressed:**
> - Equity is a residual claim with limited liability; in structural models it is option-like.
> - Technicals matter: borrow constraints, squeezes, and flow-driven markets.
> - Restructuring outcomes are uncertain; headline risk can move equity faster than credit.
>
> **Why the trade can lose money even if equity ultimately goes to zero:**
> - Mark-to-market on the short equity leg can be very adverse.
> - Borrow and funding costs can rise; margin calls can force exit.
> - **Convergence timing** dominates: you must survive to see terminal value.
>
> **Lesson:** Treat equity–credit RV as a multi-leg, path-dependent trade. Size for interim volatility, not just terminal payout.

---

## 44.8 The Risk-First Framework: Putting It Together

For any CDS relative value position, apply this systematic framework:

### 44.8.1 Step 1: Identify the Object

Explicitly state what relationship you are trading:

| Object | Description |
|--------|-------------|
| Single-name curve point | Level of 5Y CDS vs model/sector |
| CDS curve spread | 5Y-1Y steepener/flattener |
| Cash-CDS basis | Bond Z-spread vs CDS spread |
| Capital structure | Senior vs subordinated |
| Equity-credit | Equity vol vs CDS spread (Merton-based) |
| Index basis | Index vs intrinsic |

### 44.8.2 Step 2: Decompose into Exposures

For the identified trade, compute:

| Exposure | Metric |
|----------|--------|
| Spread risk (total) | Total CS01 (USD per bp) |
| Spread risk (bucketed) | Bucket CS01 by maturity |
| Default risk | JTD/VOD (USD) |
| Recovery risk | Recovery DV01 (USD per 1% R) |
| Rates risk | IR DV01 (USD per bp) |
| Time decay | Theta (USD per day) |
| Equity exposure (for Merton trades) | Equity delta, vega |
| Liquidity/technical | Qualitative assessment |
| Funding exposure | Sensitivity to repo/funding costs |

### 44.8.3 Step 3: Design Hedges

For each exposure, specify the hedge instrument and what risk it targets:

| Target | Hedge Instrument |
|--------|------------------|
| Total CS01 | Opposite CDS position |
| Bucket CS01 | Multiple CDS maturities |
| JTD | Balanced notional positions |
| IR DV01 | Interest rate swaps |
| Equity delta | Stock or equity options |
| Recovery | Often unhedgeable; scenario management |
| Funding | Funding derivatives (if available) |

### 44.8.4 Step 4: Identify Failure Modes

Before entering the trade, enumerate what can go wrong:

1. **Default event:** JTD dominates
2. **Curve shape moves:** Bucket CS01 mismatch
3. **Basis blowout:** Cash-CDS disconnection
4. **Liquidity evaporation:** Unable to exit or rebalance
5. **Recovery surprise:** Auction final price differs from assumption
6. **Roll/technical:** Index rolls, liquidity point shifts
7. **Correlation breakdown:** Equity-credit relationship breaks (Merton trades)
8. **Funding crisis:** Repo or margin costs spike (basis trades)
9. **Convergence delay:** Arbitrage takes longer than funding allows

### 44.8.5 Step 5: Define Verification Tests

Run the following scenarios before and during the trade:

| Test | Description |
|------|-------------|
| Parallel spread shock | All spreads +10bp, +50bp |
| Curve twist | Front +10bp, back +50bp; reverse |
| Default scenario | Apply VOD/JTD |
| Recovery shock | R ± 10% absolute |
| Repricing check | Linear CS01 vs full repricing |
| Funding shock | Repo +100bp, +200bp |
| Equity move (Merton) | Stock ±20% |
| Correlation break | Equity moves opposite to credit |

---

## 44.9 Worked Examples

### Example A: CS01-Neutral 5Y vs 1Y Steepener

**Setup:**
- Reference entity spreads: $S(1Y) = 80\text{bp}$, $S(5Y) = 160\text{bp}$
- Recovery: $R = 40\%$
- Buy 5Y protection: $N_5 = 10\text{mm}$
- Sell 1Y protection: notional $N_1$ to be determined

**Step 1: Compute RPV01s**

Using the credit triangle approximation $h \approx S/(1-R)$ and flat hazard per tenor:

For 1Y: $h_1 = 0.0080/0.60 = 0.0133$
$$\text{RPV01}(1) = \frac{1 - e^{-0.0133 \cdot 1}}{0.0133} \approx 0.993$$

For 5Y (with piecewise hazards): $\text{RPV01}(5) \approx 4.74$

**Step 2: Compute CS01 per 10mm**

$$\text{CS01}_{1Y} = 10\text{mm} \times 0.0001 \times 0.993 = \$993/\text{bp}$$
$$\text{CS01}_{5Y} = 10\text{mm} \times 0.0001 \times 4.74 = \$4,740/\text{bp}$$

**Step 3: Determine $N_1$ for CS01 neutrality**

$$4,740 - 993 \times \frac{N_1}{10} = 0 \implies N_1 = 47.7\text{mm}$$

**Step 4: Check JTD**

$$\text{JTD}_{5Y} = +10 \times 0.60 = +\$6.0\text{mm}$$
$$\text{JTD}_{1Y} = -47.7 \times 0.60 = -\$28.6\text{mm}$$
$$\text{Net JTD} = -\$22.6\text{mm}$$

**Conclusion:** The trade is CS01-neutral but carries $22.6\text{mm}$ short default exposure.

---

### Example B: Curve Trade P&L Scenarios

Using Example A's steepener, compute P&L under various scenarios.

**Scenario 1: Parallel +20bp**

$$\Delta V = 4,740 \times 20 + (-4,740) \times 20 = 0$$

**Scenario 2: Steepening (front +5bp, back +25bp)**

$$\Delta V = 4,740 \times 25 + (-4,740) \times 5 = 4,740 \times 20 = +\$94,800$$

**Scenario 3: Flattening (front +25bp, back +5bp)**

$$\Delta V = 4,740 \times 5 + (-4,740) \times 25 = -\$94,800$$

**Scenario 4: Default**

$$\Delta V = \text{Net JTD} = -\$22.6\text{mm}$$

The default scenario dominates all spread scenarios by two orders of magnitude.

---

### Example C: Cash-CDS Basis Trade with Financing

**Setup:**
- Bond: \$10mm face, price 102, duration 4.5, Z-spread 150bp
- CDS: 5Y spread 120bp, RPV01 = 4.7
- Basis: CDS 30bp cheap to bond
- Financing: repo at Libor + 25bp

**Step 1: Size for JTD neutrality**

Bond loss at default (price 102, recovery 40): $(1.02 - 0.40) \times 10 = \$6.2\text{mm}$

CDS notional for JTD match: $6.2 / 0.60 = \$10.33\text{mm}$

**Step 2: Compute CS01s**

Bond DV01: $4.5 \times 10.2 \times 0.0001 = \$4,590/\text{bp}$

CDS CS01: $10.33 \times 0.0001 \times 4.7 = \$4,855/\text{bp}$

**Step 3: Monthly carry**

Bond spread income: $150\text{bp} \times 10.2\text{mm} / 12 = \$12,750$

CDS premium: $120\text{bp} \times 10.33\text{mm} / 12 = \$10,330$ (paid)

Financing cost: $25\text{bp} \times 10.2\text{mm} / 12 = \$2,125$

**Net monthly carry: $12,750 - 10,330 - 2,125 = +\$295$**

**Step 4: Funding stress scenario**

If repo rises to L+150bp:

Financing cost: $150\text{bp} \times 10.2\text{mm} / 12 = \$12,750$

**Net monthly carry: $12,750 - 10,330 - 12,750 = -\$10,330$**

The trade flips from positive to deeply negative carry when funding costs spike.

---

### Example D: Senior vs Subordinated Trade

**Setup:**
- Senior 5Y CDS: $S = 120\text{bp}$, $R = 40\%$
- Sub 5Y CDS: $S = 250\text{bp}$, $R = 20\%$
- Buy sub protection: $N_{\text{sub}} = 10\text{mm}$
- Sell senior protection: notional for CS01 neutrality

**Step 1: Compute hazards and RPV01s**

Senior: $h = 0.012/0.60 = 0.02$, $\text{RPV01} \approx 4.76$

Sub: $h = 0.025/0.80 = 0.03125$, $\text{RPV01} \approx 4.63$

**Step 2: CS01 neutrality**

$$\frac{N_{\text{senior}}}{10} = \frac{4.63}{4.76} \implies N_{\text{senior}} = 9.73\text{mm}$$

**Step 3: JTD**

$$\text{JTD}_{\text{sub}} = +10 \times 0.80 = +\$8.0\text{mm}$$
$$\text{JTD}_{\text{senior}} = -9.73 \times 0.60 = -\$5.84\text{mm}$$
$$\text{Net JTD} = +\$2.16\text{mm}$$

The trade is net long default due to the different recoveries, even with CS01 neutrality.

**Step 4: Spread ratio check**

Theoretical ratio: $(1-0.20)/(1-0.40) = 0.80/0.60 = 1.33$

Market ratio: $250/120 = 2.08$

The market prices sub materially wider than the simple recovery model predicts, suggesting either (a) the market expects lower sub recovery than 20%, (b) liquidity premium in sub, or (c) the simple model is inadequate.

---

### Example E: Merton Model Calibration

**Setup (from Hull RM Example 19.4):**
- Equity value: $E_0 = \$3$ million
- Equity volatility: $\sigma_E = 80\%$
- Debt due in 1 year: $D = \$10$ million
- Risk-free rate: $r = 5\%$

**Step 1: Set up simultaneous equations**

From Merton:
$$E_0 = V_0 N(d_1) - D e^{-rT} N(d_2)$$
$$\sigma_E E_0 = N(d_1) \sigma_V V_0$$

**Step 2: Solve numerically**

Using solver (e.g., Excel Solver minimizing sum of squared residuals):

$$V_0 = \$12.40 \text{ million}$$
$$\sigma_V = 21.23\%$$

**Step 3: Compute default probability**

$$d_2 = \frac{\ln(12.40/10) + (0.05 - 0.2123^2/2)(1)}{0.2123 \sqrt{1}} = 1.14$$

$$P(\text{default}) = N(-d_2) = N(-1.14) = 12.7\%$$

**Step 4: Derive credit spread**

Market value of debt = $V_0 - E_0 = \$9.40$ million

Risk-free value of debt = $10 \times e^{-0.05} = \$9.51$ million

Expected loss = $(9.51 - 9.40)/9.51 = 1.2\%$

Implied recovery: $1 - 1.2\%/12.7\% = 91\%$

**Interpretation:** The firm has distance-to-default of 1.14 standard deviations, 12.7% default probability, and implied recovery around 91% (high because assets substantially exceed debt).

---

### Example F: Three-Leg Trade for JTD Neutrality

To achieve both CS01 and JTD neutrality, we need three instruments.

**Setup:**
- Buy 5Y protection: $N_5 = 10\text{mm}$ (given)
- Sell 3Y protection: $N_3$ (to be determined)
- Buy 1Y protection: $N_1$ (to be determined)

**Constraints:**

1. JTD neutral: $N_5 + N_3 + N_1 = 0$ (with signs for buy/sell)
2. CS01 neutral: $N_5 \cdot \text{RPV01}(5) + N_3 \cdot \text{RPV01}(3) + N_1 \cdot \text{RPV01}(1) = 0$

Using RPV01(1) = 0.99, RPV01(3) = 2.92, RPV01(5) = 4.74:

From JTD neutral: $N_1 = -10 - N_3$

Substitute into CS01:
$$10 \times 4.74 + N_3 \times 2.92 + (-10 - N_3) \times 0.99 = 0$$
$$47.4 + 2.92 N_3 - 9.9 - 0.99 N_3 = 0$$
$$37.5 + 1.93 N_3 = 0$$
$$N_3 = -19.4\text{mm}$$

Therefore: $N_1 = -10 - (-19.4) = +9.4\text{mm}$

**Result:**
- Buy 5Y: 10mm
- Sell 3Y: 19.4mm
- Buy 1Y: 9.4mm

Net notional = 0 (JTD neutral); Net CS01 $\approx$ 0. The remaining exposure is pure curve shape.

---

## 44.10 Practical Notes and Common Pitfalls

### 44.10.1 Convention Mismatches

| Pitfall | Description |
|---------|-------------|
| **CS01 sign confusion** | Different systems use different sign conventions. Always verify whether "CS01" means signed (value change) or unsigned (magnitude). |
| **Quote bump vs hazard bump** | Bumping par spreads and recalibrating gives different bucket decomposition than bumping hazard rates directly. |
| **Recovery assumption mismatch** | Bond valuations may use different recovery assumptions than CDS analytics. |
| **Day count differences** | Bond accrual and CDS accrual may use different conventions. |

### 44.10.2 Implementation Issues

1. **Survival curve interpolation:** Different interpolation schemes (linear on $-\log Q$, piecewise constant hazard) give different bucket CS01s.

2. **Accrual at default:** VOD calculations must include accrued premium; ignoring this creates small but systematic errors.

3. **Roll dates:** Index rolls and single-name CDS rolls on different schedules can create basis moves.

4. **Liquidity cliff:** 5Y is typically liquid; 4Y and 6Y may not be. Hedging curve positions requires trading at illiquid points.

5. **Merton calibration sensitivity:** Small changes in equity volatility input can produce large changes in implied default probability.

### 44.10.3 Why RV Trades Fail

The most common failure modes:

1. **Underestimating JTD:** Curve trades with large notional imbalances
2. **Funding assumptions:** Basis trades that assume stable repo funding
3. **Correlation spikes:** "Hedged" positions that become correlated in stress
4. **Liquidity evaporation:** Inability to rebalance or exit
5. **Model error:** Analytics that don't match market pricing conventions
6. **Convergence timing:** Arbitrage takes longer than funding allows (equity-credit RV, basis trades)
7. **Correlation breakdown:** Equity-credit relationship deviates from Merton predictions

---

## 44.11 Summary

CDS relative value trading requires a disciplined risk-first framework. The key points:

1. **Every RV trade carries multiple risks:** CS01 (total and bucketed), JTD, recovery sensitivity, IR DV01, theta, funding exposure, and liquidity. All must be measured and managed.

2. **CS01-neutral does not mean risk-neutral:** Curve trades can have massive JTD exposure. Index-vs-single trades carry basis risk. Capital structure trades retain recovery risk.

3. **The CDS-cash basis reflects fundamental and market factors:** Funding, delivery option, liquidity, and technicals all contribute. The basis is not pure credit mispricing.

4. **O'Kane's VOD formula captures jump-to-default risk:** $\text{VOD} = -V(t) \pm (1-R) \mp \Delta_0 S_0$. The VOD measures the "unexpected shock" of default.

5. **Three-leg trades can achieve both CS01 and JTD neutrality:** But they still carry curve shape exposure, which is the intended bet.

6. **The Merton model links equity and credit:** Equity is a call option on firm assets; equity volatility determines credit spreads through distance-to-default.

7. **Capital structure arbitrage exploits equity-credit mispricings:** But model risk, correlation breakdown, and convergence timing can cause failures.

8. **Historical and structural examples teach essential lessons:** Funding/liquidity stress and equity-credit disconnects illustrate how "hedged" positions can fail catastrophically.

9. **Failure modes must be identified before entering trades:** Default, basis blowout, funding shock, liquidity evaporation, recovery surprise, correlation breakdown.

10. **Scenario testing is mandatory:** Parallel shocks, twists, default, recovery shocks, funding shocks, and repricing checks reveal hidden exposures.

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| CS01 | Value change per 1bp spread widening | Primary spread risk metric |
| Bucket CS01 | CS01 decomposed by maturity | Reveals curve shape exposure |
| JTD/VOD | Value change on immediate default | Discontinuous risk that can dominate |
| CDS basis | CDS spread minus bond spread | Driven by multiple factors, not just credit |
| Intrinsic spread | RPV01-weighted spread of index constituents | Benchmark for index RV trades |
| Carry | Net premium accrual per period | "Curves unchanged" P&L component |
| Rolldown | MTM from maturity shortening on static curve | "Curves unchanged" P&L component |
| Recovery DV01 | Value change per 1% recovery shift | Critical for capital structure trades |
| Merton model | Equity = call option on firm assets | Links equity vol to credit spreads |
| Distance to default | Standard deviations to default barrier | Key metric for equity-credit RV |
| CreditGrades | Merton with uncertain barrier | Generates realistic short-dated spreads |

---

## Notation for This Chapter

| Symbol | Definition |
|--------|------------|
| $S(t,T)$ | Par CDS spread at time $t$ for maturity $T$ |
| $\text{RPV01}(t,T)$ | Risky PV01 from $t$ to $T$ |
| $Q(t,T)$ | Survival probability to time $T$ |
| $h(t)$ | Hazard rate at time $t$ |
| $Z(t,T)$ | Risk-free discount factor |
| $R$ | Recovery rate |
| $V(t)$ | CDS mark-to-market value |
| $N$ | Notional |
| VOD | Value on default |
| JTD | Jump-to-default (often used interchangeably with VOD) |
| $S_{\text{intrinsic}}$ | Intrinsic spread of a CDS index |
| $V_0$ | Firm value (Merton model) |
| $\sigma_V$ | Asset volatility (Merton model) |
| $E_0$ | Equity value (Merton model) |
| $\sigma_E$ | Equity volatility |
| $d_1, d_2$ | Black-Scholes parameters in Merton model |
| DD | Distance to default |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is the CDS mark-to-market formula? | $V \approx (S_{\text{market}} - S_{\text{contract}}) \cdot \text{RPV01} \cdot N$ |
| 2 | How is Credit DV01 (CS01) defined? | $-(V(S+1\text{bp}) - V(S))$, with negative sign for convention |
| 3 | Why is bucket CS01 important for curve trades? | Total CS01 can be zero while bucket exposures are large and opposite |
| 4 | What is VOD for a protection buyer? | $-V(t) + (1-R) - \Delta_0 S_0$ |
| 5 | Why can CS01-neutral curve trades have large JTD? | Notional imbalance: longer maturity needs less notional for same CS01 |
| 6 | What is the CDS-cash basis? | $S_{\text{CDS}} - S_{\text{Bond}}$ (spread measure must be specified) |
| 7 | Name three fundamental drivers of the basis | Funding, delivery option, loss on default |
| 8 | Name three market drivers of the basis | Relative liquidity, CDO technical, demand for protection |
| 9 | What is a curve steepener? | Buy long-maturity protection, sell short-maturity protection |
| 10 | What is a curve flattener? | Sell long-maturity protection, buy short-maturity protection |
| 11 | What is the senior/sub spread ratio approximation? | $S_{\text{sub}}/S_{\text{senior}} \approx (1-R_{\text{sub}})/(1-R_{\text{senior}})$ |
| 12 | Why is this ratio "very rough in practice"? | Liquidity, recovery uncertainty, APR violations, technicals |
| 13 | What is the intrinsic spread of an index? | RPV01-weighted average spread that makes sum of constituent values zero |
| 14 | What is index basis? | Index spread minus intrinsic spread |
| 15 | What is "carry" in a CDS position? | Net premium accrual under curves unchanged |
| 16 | What is "rolldown" in a CDS position? | MTM change from maturity shortening on static curve |
| 17 | How many instruments to achieve CS01 and JTD neutrality? | Three (two constraints, three unknowns) |
| 18 | What is the biggest failure mode in curve trades? | JTD dominance: default produces outsized loss |
| 19 | Why do basis trades fail? | Funding blowout, correlation breakdown, JTD mismatch |
| 20 | What is the key test for any RV position? | Scenario suite: parallel, twist, default, recovery, repricing |
| 21 | What is the mean recovery for senior unsecured bonds (Altman data)? | 34.89% (median 42.27%) |
| 22 | What is the mean recovery for subordinated bonds (Altman data)? | 29.03% (median 31.96%) |
| 23 | Why is VOD called a measure of "unexpected shock"? | If spreads widen gradually before default, V(t) already reflects default risk, so VOD is small |
| 24 | What determines the sign of theta for a CDS position? | Short protection (receiving premium) has positive theta; long protection has negative theta |
| 25 | What is Recovery DV01? | Change in MTM for 1% absolute change in recovery rate assumption |
| 26 | What is capital structure arbitrage (equity-credit)? | Exploiting mispricings between equity derivatives and CDS based on Merton framework |
| 27 | What is the Merton equity formula? | $E = V N(d_1) - D e^{-rT} N(d_2)$ — equity is a call on firm assets |
| 28 | What is distance-to-default? | Number of standard deviations firm value must fall to trigger default: $d_2$ |
| 29 | What is CreditGrades? | Extension of Merton with uncertain default barrier, generates short-dated spreads |
| 30 | What is the LBO trade? | Buying protection anticipating leverage event that widens spreads |
| 31 | Why did capital structure arbitrage fail for some? | Model risk, correlation breakdown, jump risk, convergence timing |
| 32 | Why can basis trades fail in stress? | Funding/haircut shocks and bond illiquidity can dominate even if the CDS hedge behaves as designed |
| 33 | How does convertible arb affect the CDS basis? | Creates natural buying of CDS protection (to hedge converts), can tighten basis |
| 34 | What is event risk in credit? | Risk of M&A, LBO, or other corporate action that affects credit quality |
| 35 | How does equity volatility relate to credit spreads (Merton)? | $\sigma_E E = N(d_1) \sigma_V V$ — higher equity vol implies higher asset vol and wider spreads |
| 36 | How can equity-credit RV fail? | Equity can rally/squeeze while credit stays distressed; timing and margin/funding can force exit |

---

## Mini Problem Set

**1.** A 5Y CDS has RPV01 = 4.5, a 2Y CDS has RPV01 = 1.9. For a 5Y vs 2Y steepener with $N_5 = 10\text{mm}$ buy protection, what $N_2$ achieves CS01 neutrality?

> **Solution sketch:** $N_2 = 10 \times 4.5 / 1.9 = 23.7\text{mm}$ sell protection.

**2.** Using Problem 1, compute net JTD assuming $R = 40\%$.

> **Solution sketch:** JTD$_5 = +6.0\text{mm}$, JTD$_2 = -14.2\text{mm}$, Net = $-8.2\text{mm}$ (short default).

**3.** A bond trades at 98 with Z-spread 180bp. The 5Y CDS trades at 150bp. What is the basis?

> **Solution sketch:** Basis = 150 - 180 = -30bp (CDS tighter than bond, negative basis).

**4.** Explain why a negative basis trade (buy bond, buy CDS protection) can fail even if the basis converges.

> **Solution sketch:** Funding costs may exceed the carry from basis convergence; liquidity shock may prevent rolling repo; default may occur with JTD mismatch.

**5.** Senior CDS trades at 100bp ($R=40\%$), subordinated at 200bp. What recovery assumption for sub makes the theoretical ratio match?

> **Solution sketch:** $200/100 = 2 = (1-R_{\text{sub}})/0.60 \implies R_{\text{sub}} = 1 - 1.2 < 0$. Impossible, implying the market prices additional risks beyond simple recovery.

**6.** For a three-leg trade with $N_1 = 8$, $N_3 = -20$, $N_5 = 12$ (positive = buy protection), verify JTD neutrality and compute the bucket exposure profile.

> **Solution sketch:** Net notional = $8 - 20 + 12 = 0$ (JTD neutral). Bucket exposures depend on RPV01 contributions by tenor; the trade is a curve shape bet.

**7.** Why does O'Kane say VOD measures "unexpected shock"?

> **Solution sketch:** If spreads widen gradually before default, V(t) approaches $(1-R)$ for protection buyer, so VOD approaches zero. Large VOD means default was a surprise relative to prior MTM.

**8.** Design a scenario test suite for a senior-sub compression trade.

> **Solution sketch:** (1) Parallel spread shock both tiers; (2) Sub widens more than senior (decompression); (3) Default with both recovery assumptions; (4) Recovery shock on sub only; (5) Liquidity scenario where sub becomes untradeable.

**9.** An index has 100 names. The simple average spread is 80bp, but the intrinsic spread is 95bp. What does this imply about the spread distribution?

> **Solution sketch:** The intrinsic spread being higher implies that wider-spread names (with higher RPV01 weights) are pulling up the weighted average. There is likely significant dispersion with a few names trading much wider than the median.

**10.** A trader has a 5Y CDS position with CS01 = \$5,000/bp. The theta is +\$200/day. How many days of carry equals a 1bp parallel spread move?

> **Solution sketch:** 1bp move = \$5,000. Days to offset = 5,000 / 200 = 25 days. The position needs 25 days of positive theta to compensate for a 1bp spread widening.

**11.** A firm has equity value $E_0 = \$5$ million, equity volatility $\sigma_E = 60\%$, debt $D = \$20$ million due in 2 years, risk-free rate 4%. Using the Merton model approach, explain how you would find $V_0$ and $\sigma_V$.

> **Solution sketch:** Set up two equations: (1) $E_0 = V_0 N(d_1) - D e^{-rT} N(d_2)$ and (2) $\sigma_E E_0 = N(d_1) \sigma_V V_0$. Solve numerically using iterative solver. The solution gives firm value and asset volatility consistent with observed equity.

**12.** Using Merton, if $V_0 = \$25$ million, $D = \$20$ million, $\sigma_V = 25\%$, $T = 2$ years, $r = 4\%$, compute the distance-to-default.

> **Solution sketch:** $d_2 = \frac{\ln(25/20) + (0.04 - 0.25^2/2)(2)}{0.25\sqrt{2}} = \frac{0.223 + 0.0175}{0.354} = 0.68$. Distance-to-default is 0.68 standard deviations.

**13.** In Problem 12, what is the implied default probability?

> **Solution sketch:** $P(\text{default}) = N(-0.68) \approx 25\%$. This is a high-risk firm.

**14.** Equity vol is 50%, CDS spread is 200bp. The Merton model implies credit spread should be 150bp. Is equity cheap or rich relative to credit? What trade do you put on?

> **Solution sketch:** CDS is 50bp wider than Merton implies. Either equity vol is too low (equity puts are cheap) or CDS is too wide (CDS protection is expensive). Trade: buy equity puts, sell CDS protection, expecting convergence.

**15.** A basis trade is entered at -30bp negative basis. Over the next month, repo funding goes from L+25bp to L+175bp. The basis remains at -30bp. Explain the P&L impact.

> **Solution sketch:** Monthly carry shifts: previously earning +\$X from basis minus low funding; now losing due to high funding. Even though basis hasn't moved, the trade becomes unprofitable due to funding cost increase. This is the funding-stress failure mode.

**16.** In a bankruptcy situation, equity can rally sharply even while bonds trade at distressed levels. Explain why a trader who shorted equity and sold CDS protection might have lost money, even if equity is eventually wiped out.

> **Solution sketch:** The equity rally creates mark-to-market losses on the short. Margin calls, borrow costs, and risk limits can force exit before terminal convergence. The "arbitrage" can be correct on terminal value but untradeable due to interim MTM and financing constraints (timing risk).

---

## References

- Dominic O'Kane, *Modelling Single-name and Multi-name Credit Derivatives* (CDS risk measures, basis, indices)
- John C. Hull, *Risk Management and Financial Institutions* (structural model intuition and credit risk mechanics)
