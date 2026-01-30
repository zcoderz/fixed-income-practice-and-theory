# Chapter 35: Default, Recovery, and Credit Events — Economic vs Contractual Reality

---

## Introduction

When a credit analyst says "XYZ Corp has defaulted," what exactly does that mean? The answer is less obvious than it first appears—and getting it wrong can be extraordinarily expensive.

Consider this scenario: a trader holds $50 million notional of CDS protection on a distressed issuer. The reference entity's bonds have collapsed from 95 to 35, equity has cratered, and the financial press is full of restructuring speculation. The trader expects a windfall from the CDS protection. But weeks pass, and no payment arrives. The contract hasn't triggered because no **credit event**—the precise contractual definition that activates CDS protection—has yet occurred. Economic distress is not the same as a contractual trigger.

This distinction between economic default (what the market perceives) and contractual credit event (what triggers payments) lies at the heart of credit derivatives. O'Kane emphasizes that "the credit event is the legal term for the event which triggers the payment of the protection leg. It is an event which is similar to default but, as we will see, it is not exactly the same as the event of default as defined by the rating agencies."

This chapter establishes the foundational concepts for credit derivatives pricing:

1. **Economic Default vs Contractual Credit Event** — Why spreads can blow out while CDS contracts sit dormant
2. **Par vs Distressed Trading Conventions** — How the market quotes instruments changes as credit deteriorates
3. **Recovery Rates and Loss Given Default** — How much you get back, and why it's not a fixed number
4. **Credit Risk Premium Decomposition** — Why market spreads exceed actuarial expectations
5. **Credit Event Taxonomy** — The ISDA definitions that determine when protection pays
6. **Structural Model Intuition** — The Merton model preview connecting equity and credit

Understanding these concepts is essential before tackling CDS mechanics (Chapter 38), CDS pricing (Chapter 41), survival curve construction (Chapter 36), or credit relative value (Chapter 44). Every formula in those chapters builds on the definitions established here.

---

## Conventions and Notation

| Symbol | Definition |
|--------|------------|
| $N$ | CDS notional (dollars) |
| $R$ | Recovery rate as a fraction of par (dimensionless, 0–1) |
| $\text{LGD}$ | Loss given default $= 1 - R$ (dimensionless) |
| $P$ | Observed distressed/defaulted bond price per 100 par (price points) |
| $S$ | CDS spread as annual rate in decimal (e.g., 200 bp = 0.02) |
| $\lambda$ | Hazard rate (instantaneous default intensity) |
| $\alpha$ | Day-count fraction from last premium date to default date (Actual/360) |
| $\text{CR}$ | Coverage ratio = Market spread / Actuarial spread |

**Sign convention:** Payoffs are written from the perspective of the protection buyer unless stated otherwise.

---

## 35.1 Economic Default vs Contractual Credit Event

### The Distinction That Costs Money

Traders often use "default" loosely, but O'Kane warns that "in the credit derivatives market and in this book, the term 'default' is often used when the precise term 'credit event' is meant." This conflation obscures a crucial distinction:

**Economic default** is a continuous deterioration visible in market prices. Spreads widen, equity falls, bond prices collapse, funding dries up. This is what traders watch on their screens.

**Contractual credit event** is a discrete boundary crossing defined by legal documentation. Your CDS protection pays only when a defined credit event occurs, the notice/evidence conditions are satisfied, and the settlement steps complete.

The mismatch creates **timing risk** for hedgers. A bond-plus-CDS hedge can show substantial P&L volatility during the "gray zone" where economic distress is obvious but no contractual trigger has occurred.

> **Analogy: The Fire Alarm**
>
> *   **Economic Default (Smoke)**: The room is filled with smoke. Everyone *knows* there is a fire. The price of the house (bonds) has collapsed.
> *   **Credit Event (Sprinklers)**: The sprinklers (CDS Payout) only turn on if someone pulls the specific red handle on the wall.
> *   **The Risk**: You might die of smoke inhalation (lose money on the bond) while standing right next to the sprinkler system, just because the handle hasn't been pulled yet (no legal "Failure to Pay").

### 35.1.1 Par vs Distressed Trading Conventions

As credit quality deteriorates, the way the market quotes instruments undergoes a fundamental shift. Understanding this transition is critical for interpreting prices and managing positions through distress.

**Par Trading (Investment Grade and Stable High Yield)**

When bonds trade near par (typically 90–105), the market focuses on **yield and spread**. Investors ask: "Will I get my coupon? What spread am I earning over the risk-free rate?" The relevant metrics are yield-to-maturity, G-spread, I-spread, and Z-spread. CDS trades at a running spread quoted in basis points.

**Distressed Trading (Stressed Credits)**

When bonds collapse to 20–50, the trading paradigm shifts entirely. The market now focuses on **recovery value**—"What is the liquidation value of the assets? How much will bondholders receive in bankruptcy?" Bonds stop trading on yield and start trading on "cents on the dollar" or "points."

> **Desk Reality: The Distressed Quote Shift**
>
> A trader calling for a quote on a stressed name will hear very different language:
> - *Par trading*: "I'm bid at 250 over, offered at 245"
> - *Distressed trading*: "I'm bid at 35 cents, offered at 37"
>
> **Rule of thumb (not a hard rule):** once spreads are extremely wide and prices are far below par, many desks stop talking in yield/spread language and start talking in price/recovery language ("cents on the dollar"). At that point, yield-to-maturity can become a poor guide to economics: a bond trading at 30 cents with a 5% coupon has a "yield" that is mathematically computable but not operationally useful for distressed decision-making.

**CDS Upfront Convention for Distressed Names**

For distressed credits, CDS also shifts from running spread to upfront pricing. A standardized coupon (often 100 bp for investment grade and 500 bp for high yield in North America, depending on contract terms) remains fixed, and the market quotes the upfront payment required to enter the contract.

| Quoting Regime | Bond Quote | CDS Quote | Investor Focus |
|----------------|------------|-----------|----------------|
| Par Trading | Yield/Spread (bps) | Running spread (bps) | Coupon, spread duration |
| Distressed | Price (cents on dollar) | Upfront (% of notional) | Recovery, timing to default |

The upfront amount $U$ relates to the par spread $S$ and standard coupon $C$ approximately as:

$$U \approx (S - C) \times \text{Risky Duration}$$

When spreads are 2000 bp and the standard coupon is 500 bp, the protection buyer pays a significant upfront premium to enter protection at the below-market coupon rate.

> **Practitioner Note:** The distressed CDS market quotes upfront as a percentage of notional. A quote of "50 points up" means the protection buyer pays 50% of notional upfront plus the running 500 bp coupon. This is economically equivalent to buying protection at the par spread, but the upfront convention standardizes cash flows and reduces counterparty exposure.

### 35.1.2 What Constitutes a Default?

Moody's Investor Services provides the most commonly used definition. O'Kane summarizes: "Moody's Investor Services defines default as 'any missed or delayed payment of interest or principal, bankruptcy, receivership or distressed exchange where (i) the issuer offered bondholders a new security or package of securities that amount to a diminished financial obligation (such as preferred or common stock or debt with a lower coupon or par amount) or (ii) the exchange had the apparent purpose of helping the borrower avoid default.'"

A **technical default**—a failure to pay that is quickly rectified—is typically not included in rating agency default statistics. But it may still trigger a CDS contract, creating basis between cash and derivatives. O'Kane notes that "if a failure to pay principal or coupon occurs as a result of some omission which is quickly rectified, then the event is known as technical default. Such an occurrence is not usually included in rating agency default statistics."

### 35.1.3 Historical Default Rates

Before modeling credit derivatives, practitioners need empirical grounding in how often defaults occur. O'Kane presents cumulative default rates from Moody's covering 1983–2005:

| Rating | 1-Year (%) | 5-Year (%) | 10-Year (%) |
|--------|------------|------------|-------------|
| Aaa | 0.0 | 0.1 | 0.2 |
| Aa | 0.0 | 0.2 | 0.4 |
| A | 0.0 | 0.6 | 1.2 |
| Baa | 0.2 | 2.2 | 4.5 |
| Ba | 1.3 | 11.1 | 17.8 |
| B | 5.7 | 25.1 | 32.1 |
| Caa-C | 21.0 | 41.2 | 43.3 |

These historical default rates are used to calibrate risk models, but they are **not** used directly for pricing. Pricing requires the risk-neutral framework, where implied default probabilities are extracted from market spreads (covered in Chapter 36).

> **Why historical ≠ risk-neutral:** Hull presents extensive evidence that hazard rates implied from bond spreads are typically much higher than historical default rates. For example, seven-year hazard rates for Baa-rated bonds are approximately 5× higher when implied from spreads versus calculated from historical data. This "default risk premium" compensates investors for: (1) uncertainty in the historical statistics, (2) the systematic risk that defaults concentrate in bad economic times, and (3) the difficulty of diversifying the highly skewed payoffs of credit instruments.

Hull explains the core reason: "By far the most important reason for the results... is that bonds do not default independently of each other. There are periods of time when default rates are very low and periods of time when they are very high." This clustering creates systematic risk that cannot be diversified away, requiring a risk premium.

> **Concept: The Hazard Rate ($\lambda$)**
>
> In credit modeling, we don't just ask "Will it default?" we ask "How intense is the danger right now?"
> *   **$\lambda$ (Hazard Rate)**: The instantaneous probability of default.
> *   **Analogy**: A Geiger counter clicking. High $\lambda$ = lots of clicks (high danger).
> *   **Triangle Rule of Thumb**:
>     $$ \text{Credit Spread} \approx (1 - \text{Recovery}) \times \lambda $$
>     If Recovery is 40% and Spread is 600bps (6%), then $\lambda \approx 10\%$. The market expects a 10% chance of default this year.

---

## 35.2 Recovery Rates: What You Get Back

### 35.2.1 Defining Recovery

Hull provides the canonical definition: "The recovery rate for a bond is normally defined as the bond's market value shortly after default, as a percent of its face value."

More precisely, recovery rate $R$ is:

$$\boxed{R = \frac{\text{Bond price shortly after default}}{\text{Face value}} = \frac{P}{100}}$$

If a defaulted bond trades at $35 per $100 face, then $R = 0.35$.

**Loss given default (LGD)** is the complement:

$$\boxed{\text{LGD} = 1 - R}$$

The LGD is what the protection seller pays (or equivalently, what the protection buyer receives) as a fraction of notional.

### 35.2.2 Recovery Rate Statistics by Seniority

O'Kane presents empirical recovery data from Altman et al. (2003b), based on bond prices just after default and loan prices 30 days after default:

| Debt Seniority | Type | Number of Issues | Median Recovery (%) | Mean Recovery (%) | Std Dev (%) |
|----------------|------|------------------|---------------------|-------------------|-------------|
| Senior secured | Loans | 155 | 73.00 | 68.50 | 24.4 |
| Senior unsecured | Loans | 28 | 50.50 | 55.00 | 28.4 |
| Senior secured | Bonds | 220 | 54.49 | 52.84 | 23.1 |
| Senior unsecured | Bonds | 910 | 42.27 | 34.89 | 26.6 |
| Senior subordinated | Bonds | 395 | 32.35 | 30.17 | 25.0 |
| Subordinated | Bonds | 248 | 31.96 | 29.03 | 22.5 |

O'Kane highlights several key observations from this data:

1. **Seniority is the dominant driver.** "The most important driver of the recovery rate is the position of the debt in the capital structure of the firm, i.e. senior debt recovers more on average than subordinated debt."
2. **Loans recover more than bonds.** "The recovery rate of loans exceeds that of bonds. This is because the loans typically contain additional covenants."
3. **Recovery is highly variable.** Standard deviations of 22–28% mean individual outcomes can diverge sharply from means. O'Kane notes that "the absolute priority rule (APR) is not always obeyed in the US, meaning that in certain circumstances, holders of subordinated debt may recover more than holders of more senior debt."
4. **The CDS-relevant benchmark is senior unsecured bonds** at approximately 35% mean recovery (34.89% precisely). Hull notes that "an average recovery rate that is often assumed is 40%."

### 35.2.3 Absolute Priority Rule and Violations

The **Absolute Priority Rule (APR)** is the legal principle that senior creditors must be paid in full before junior creditors receive anything. In theory, the waterfall proceeds as:

1. Secured creditors (up to collateral value)
2. Senior unsecured creditors
3. Subordinated creditors
4. Preferred equity
5. Common equity

In practice, as O'Kane notes, APR violations occur regularly in U.S. bankruptcies. Several mechanisms drive these violations:

**Negotiated Settlements:** Bankruptcy is expensive and time-consuming. Senior creditors may accept less than full recovery to avoid protracted litigation, especially when junior creditors threaten to delay proceedings.

**Option Value of Equity:** In Chapter 11 reorganizations, equity holders retain some claims on the restructured entity. The option value of potential recovery gives them bargaining power.

**Timing and Liquidity:** Senior creditors may prefer quick, certain recovery over uncertain claims that could take years to resolve.

> **Desk Reality: APR Violations in Practice**
>
> A distressed debt trader knows that textbook recovery waterfalls rarely match reality. Common patterns include:
> - Equity receives 5–10% of reorganized equity even when senior unsecured recovers only 60 cents
> - Subordinated bondholders receive recovery when seniors are impaired
> - "Gifting" arrangements where seniors share recovery to expedite resolution
>
> This means subordinated recovery rates in the historical data (29–32%) are higher than a strict APR analysis would predict.

### 35.2.4 Recovery vs Default Rate Correlation

One of the most important empirical findings is that recovery rates are **negatively correlated** with default rates. O'Kane explains: "When macroeconomic default rates increase, the average recovery rate falls, and vice versa."

The mechanism is supply and demand in distressed debt markets. O'Kane elaborates: "The dynamic is that when there is a high default rate, there is an oversupply of defaulted assets in the distressed debt market. As a result, the price of distressed debt falls. The opposite occurs when we are in a period of below average default rates."

Altman et al. (2003a) found an $R^2$ of 0.51 for the relationship, with a best-fit line of:

$$\text{Recovery Rate} \approx 0.51 - 2.6 \times \text{Default Rate}$$

Hull confirms this pattern: "In a year when the number of bonds defaulting is low, economic conditions are usually good and the average recovery rate on those bonds that do default might be as high as 60%; in a year when the default rate on corporate bonds is high, economic conditions are usually poor and the average recovery rate on the defaulting bonds might be as low as 30%."

This correlation is **systematic risk** that affects portfolio credit products like CDOs. It means a bad default year is "doubly bad" for credit portfolios—more defaults occur, and each default recovers less.

### 35.2.5 Credit Derivatives Recovery vs Workout Recovery

O'Kane cautions practitioners to distinguish between two recovery concepts:

**Credit derivatives recovery price:** The price of the reference obligation determined within approximately 72 days of the credit event (via auction or dealer poll). This is what determines CDS payoffs.

**Workout recovery:** The ultimate amount received by bondholders after the full bankruptcy/liquidation process, which can take years.

These can differ significantly. O'Kane notes: "Significant differences might occur if new information arrives between the setting of the recovery price and the completion of the workout process. The recovery price is also vulnerable to supply and demand effects in the distressed debt market."

> **Desk Reality: CDS vs Workout Recovery Divergence**
>
> Consider a company that defaults with CDS auction recovery of 25 cents. Over the next 3 years, the bankruptcy process unfolds and bondholders ultimately receive 45 cents in a combination of new debt, equity, and cash.
>
> The CDS protection buyer received $(1 - 0.25) = 75\%$ of notional at the auction. The bondholder who held through workout received only $(1 - 0.45) = 55\%$ loss. The CDS overcompensated by 20 points.
>
> Conversely, if new negative information emerges post-default (fraud, environmental liabilities), workout recovery could be lower than the auction price.

For CDS pricing, the credit derivatives recovery price is the relevant concept.

---

## 35.3 Credit Risk Premium: Why Spreads Exceed Actuarial Expectations

### 35.3.1 The Risk Premium Puzzle

Market credit spreads consistently exceed what pure actuarial default expectations would imply. Hull documents that risk-neutral hazard rates implied from spreads are typically 2–10× higher than historical default rates. This "excess spread" decomposes into several components:

$$\boxed{S_{\text{market}} = S_{\text{actuarial}} + \pi_{\text{default}} + \pi_{\text{recovery}} + \pi_{\text{liquidity}}}$$

where:
- $S_{\text{actuarial}} = \lambda_{\text{historical}} \times (1 - R)$ is the expected loss component
- $\pi_{\text{default}}$ is the default risk premium (compensation for systematic default risk)
- $\pi_{\text{recovery}}$ is the recovery uncertainty premium
- $\pi_{\text{liquidity}}$ is the liquidity premium for illiquid credit instruments

### 35.3.2 Coverage Ratio: Quantifying the Risk Premium

The **coverage ratio** measures how much compensation the market demands relative to actuarial expectations:

$$\boxed{\text{Coverage Ratio} = \frac{S_{\text{market}}}{S_{\text{actuarial}}} = \frac{S_{\text{market}}}{\lambda_{\text{historical}} \times (1 - R)}}$$

Hull's data shows coverage ratios that increase dramatically for lower-rated credits:

| Rating | Historical Default Rate (7yr) | Implied Hazard (7yr) | Coverage Ratio |
|--------|------------------------------|---------------------|----------------|
| Aaa | 0.04% | 0.60% | ~15× |
| Aa | 0.06% | 0.67% | ~11× |
| A | 0.13% | 0.99% | ~8× |
| Baa | 0.47% | 2.25% | ~5× |
| Ba | 1.93% | 4.15% | ~2× |
| B | 4.73% | 6.54% | ~1.4× |

The pattern is instructive: investment-grade credits command enormous coverage ratios (the market demands 5–15× actuarial expectations), while speculative-grade credits have more modest ratios. This reflects:

1. **Systematic risk premium:** Defaults cluster in recessions when marginal utility is high
2. **Uncertainty premium:** Less data on rare investment-grade defaults
3. **Liquidity premium:** Corporate bonds are less liquid than government bonds

> **Desk Reality: Trading the Coverage Ratio**
>
> Some credit investors use coverage ratio as a relative value metric:
> - *High coverage ratio* (market spread >> actuarial): Expensive protection, or compensation for risks not in historical data
> - *Low coverage ratio* (market spread ≈ actuarial): Cheap protection, or market complacency about tail risks
>
> During the 2007 pre-crisis period, coverage ratios on structured credit fell below 1.5×—the market was barely compensating for actuarial losses, let alone risk premia. This was a warning sign.

### 35.3.3 Why Historical ≠ Risk-Neutral

Hull explains the fundamental reason why we cannot use historical statistics directly for pricing: "By far the most important reason for the results... is that bonds do not default independently of each other. There are periods of time when default rates are very low and periods of time when they are very high."

This correlation has profound implications:

1. **Diversification is limited.** A portfolio of 100 credits doesn't reduce systematic default risk the way a portfolio of 100 independent bets would.

2. **Risk-neutral measure differs.** Under the risk-neutral measure, default intensities are higher because we must price in the systematic risk premium.

3. **Recovery uncertainty compounds.** The negative correlation between default rates and recovery rates (Section 35.2.4) means that when many defaults occur, each default is more painful.

---

## 35.4 Credit Event Taxonomy: The ISDA Definitions

### 35.4.1 Hard vs Soft Credit Events

CDS contracts define specific events that trigger protection payments. O'Kane explains that "the market generally divides these into hard and soft credit events."

**Hard credit events** cause all debt to become immediately due and payable, typically trading at a single distressed price. O'Kane describes these as events that "would cause all the debt of the reference entity to become immediately due and payable and hence trade at the same price. It is what the rating agencies would traditionally call default."

| Credit Event | Description |
|--------------|-------------|
| **Bankruptcy** | Corporate becomes insolvent or is unable to pay its debts. Not relevant for sovereign issuers. |
| **Failure to Pay** | Failure to make due payments, taking into account a grace period. |
| **Obligation Acceleration** | Obligations become due earlier than scheduled due to default. Used mostly in emerging market contracts. |
| **Obligation Default** | Obligations become due and payable prior to maturity. Rarely used. |
| **Repudiation/Moratorium** | Reference entity or government authority rejects or challenges the validity of obligations. Used for emerging market sovereigns. |

**Soft credit events** allow debt to continue trading with a term structure of prices:

| Credit Event | Description |
|--------------|-------------|
| **Restructuring** | Changes in debt obligations associated with credit deterioration. Post-event, short-dated bonds typically trade higher than long-dated bonds. |

O'Kane emphasizes that "restructuring is the only soft credit event. Following a restructuring event, debt can continue trading with a term structure of prices. Short dated assets will tend to trade at higher prices than long dated assets, and assets with higher coupons will trade with a higher price than those with lower coupons."

### 35.4.2 Credit Event Timeline: From Distress to Settlement

Understanding the timeline from initial distress to final CDS settlement is critical for hedgers and traders managing positions through credit events. The process can take months and involves multiple parties.

**Phase 1: Pre-Event Deterioration (Weeks to Months)**

| Day | Event | Market Impact |
|-----|-------|---------------|
| T-90 | Rating downgrade, covenant breach | Spreads widen, bond prices fall |
| T-60 | Missed earnings, liquidity concerns | CDS spread > 500bp, bonds < 80 |
| T-30 | Restructuring talks, advisor hired | Spread > 1000bp, bonds < 60 |
| T-7 | Grace period begins (missed payment) | Distressed levels, high uncertainty |
| T | Grace period expires → **Credit Event** | Protection triggered |

**Phase 2: Credit Event Determination (0–14 days)**

| Day | Event | Participant |
|-----|-------|-------------|
| T+0 | Credit event occurs | Reference entity |
| T+1 to T+3 | Credit Event Notice submitted | Protection buyer or dealer |
| T+3 to T+14 | ISDA Determinations Committee convenes | DC members vote |
| T+14 | DC publishes determination | ISDA |

**Phase 3: Auction and Settlement (14–60 days)**

| Day | Event | Participant |
|-----|-------|-------------|
| T+14 to T+30 | Auction date announced | ISDA/auction administrator |
| T+30 | Initial market midpoint (IMM) | Participating dealers |
| T+30 | Open auction: limit orders | All market participants |
| T+30 | Final price determined | Auction mechanism |
| T+33 to T+60 | Cash settlement | CDS counterparties |

> **Desk Reality: The "Gray Zone" Problem**
>
> Between T-30 and T+30, positions are in limbo:
> - Bond prices have collapsed, but CDS hasn't paid yet
> - Mark-to-market shows large unrealized losses on bonds, gains on CDS
> - If you're hedged, P&L should be flat—but accountants may not agree
> - Funding desks may margin the bond position based on current price
>
> Traders managing this period face basis risk, funding risk, and operational risk simultaneously.

### 35.4.3 The ISDA Determinations Committee

Since 2009, credit event determinations for standard CDS contracts are governed by **ISDA Determinations Committees (DC)**—regional panels of 15 dealer and buy-side members who vote on whether a credit event has occurred.

The DC process standardizes what was previously bilateral negotiation:

1. **Request for determination:** Any market participant can request the DC to rule on a potential credit event
2. **Deliberation:** The DC reviews publicly available information and votes
3. **Supermajority required:** Credit event determination requires 12/15 (80%) votes
4. **Binding result:** DC decisions bind all standard CDS contracts referencing the entity

> **Practitioner Note:** The DC system was created after the 2008 crisis when bilateral disputes over credit events (especially restructuring) caused settlement delays and legal uncertainty. The trade-off: individual market participants cede control over their own credit event determinations to a committee that may have different economic interests.

### 35.4.4 Restructuring Clauses: Regional Variations

The treatment of restructuring as a credit event has created persistent regional differences in CDS contracts. O'Kane provides the taxonomy:

| Clause | Abbreviation | Description |
|--------|--------------|-------------|
| **Old-Restructuring** | Old-Re (OR) | Original standard with 30-year maximum maturity on deliverables |
| **Modified-Restructuring** | Mod-Re (MR) | US standard. Limits deliverable maturities following restructuring |
| **Modified-Modified-Restructuring** | Mod-Mod-Re (MMR) | European standard. Slightly broader deliverable range than Mod-Re |
| **No-Restructuring** | No-Re (NR) | Removes restructuring as a credit event entirely |

The choice of restructuring clause affects CDS spreads because it affects the value of the **delivery option**—the protection buyer's right to choose which deliverable to surrender. O'Kane shows the spread ordering:

$$S_{\text{Old-Re}} > S_{\text{Mod-Mod-Re}} > S_{\text{Mod-Re}} > S_{\text{No-Re}}$$

The CDX (North American) indices use No-Re, while iTraxx (European) indices include restructuring. This creates basis between regional index products.

---

## 35.5 The CDS Payoff Structure

### 35.5.1 Protection Leg: What Gets Paid at Credit Event

The protection leg is the contingent payment made by the protection seller to compensate the protection buyer for losses. Under cash settlement:

$$\boxed{\Pi_{\text{prot}} = N(1 - R) = N \times \text{LGD}}$$

where $N$ is the notional and $R$ is the recovery rate (settlement price divided by par).

**Unit check:** $N$ is dollars, $(1-R)$ is dimensionless, so $\Pi_{\text{prot}}$ is dollars. ✓

**Boundary check:** If $R = 1$ (full recovery), payout is zero. If $R = 0$ (total loss), payout is full notional. ✓

### 35.5.2 The ISDA Auction Mechanism

Since 2005, most CDS contracts settle via a standardized ISDA auction process rather than bilateral physical settlement. Hull describes this process: "an ISDA-organized auction process is used to determine the mid-market value of the cheapest deliverable bond several days after the credit event."

**Stage 1: Initial Market Midpoint (IMM)**

Participating dealers submit two-way markets (bid and offer) on deliverable obligations. The midpoint of the inside market establishes the **Initial Market Midpoint (IMM)**—a preliminary recovery estimate.

**Stage 2: Open Auction**

The open auction determines the final recovery price:

1. **Physical settlement requests:** Counterparties wanting physical settlement submit requests specifying direction (buy or sell bonds at the final price) and size
2. **Net open interest:** Requests are netted to determine if there's a net buy or sell imbalance
3. **Limit orders:** Market participants submit limit orders to fill the imbalance
4. **Crossing:** Orders cross at the price that clears the imbalance, subject to caps vs the IMM
5. **Final price:** This becomes the recovery rate $R$ for all cash-settled contracts

> **Desk Reality: Auction Dynamics**
>
> The auction creates predictable trading patterns:
> - Before auction: Distressed debt traders accumulate deliverables anticipating demand from protection buyers who want physical settlement
> - IMM submission: Dealers provide tight markets; their submissions affect the preliminary price
> - Open auction: Large open interest can move the final price significantly from IMM
> - Post-auction: Deliverables trade toward the final price; basis collapses
>
> In Lehman (2008), the net open interest was to sell $4.9 billion of bonds at the final price of 8.625%—one of the largest auction settlements in history.

### 35.5.3 Physical vs Cash Settlement Economics

Under **physical settlement**, the protection buyer delivers face value of deliverable obligations and receives par in cash.

Under **cash settlement**, the protection seller pays par minus the recovery/settlement price, determined by auction.

These are economically equivalent when the settlement price equals the market price of deliverables. Let the post-event bond price be $P$ per 100 par, so $R = P/100$:

**Cash settlement gain:**
$$\Pi = N(1 - R) = N \times \frac{100 - P}{100}$$

**Physical settlement gain:**
$$\text{Buy deliverable at } P, \text{ deliver for } 100 \Rightarrow \text{Gain} = N \times \frac{100 - P}{100}$$

The equivalence holds under idealized conditions. In practice, physical settlement creates a **delivery option**: the protection buyer chooses which deliverable to surrender, selecting the cheapest-to-deliver.

### 35.5.4 The Delivery Option

O'Kane illustrates with an example: "Consider a protection buyer holding $100 face value of an asset which trades at a price of $43 and $100 of protection. Suppose that a restructuring credit event occurs and that another deliverable asset is trading at a price of $37. To maximise his profit, the protection buyer can sell the original asset for $43, buy the cheaper asset for $37, and deliver that for par, making an additional profit of $6 from the delivery option."

The delivery option is most valuable after soft credit events (restructuring) where different bonds trade at different prices.

### 35.5.5 Premium Accrual at Default

CDS contracts specify that the protection buyer pays accrued premium from the last payment date to the default date:

$$\boxed{\Pi_{\text{accrued prem}} = N \times S \times \alpha}$$

where $S$ is the annual spread (decimal) and $\alpha$ is the day-count fraction (typically Actual/360).

**Worked Example:**
- Notional $N = \$10,000,000$
- CDS spread $S = 300$ bp $= 0.03$
- Days since last premium: 50 days
- Accrual fraction: $\alpha = 50/360 = 0.1389$

$$\Pi_{\text{accrued}} = 10{,}000{,}000 \times 0.03 \times 0.1389 = \$41{,}667$$

This accrued premium is paid by the protection buyer at default, in addition to any scheduled payments already made.

> **Basis note:** CDS contracts pay accrued premium on default, but bondholders' accrued coupon can be lost in default. O'Kane notes this asymmetry: "Following a credit event, a CDS pays the protection seller the premium which has accrued since the previous payment date. However, when a bond defaults, the owner's claim is on the face value, and so any accrued coupons are lost. The effect on the inclusion of coupon accrued on default is to lower the CDS spread and so reduce the basis."

---

## 35.6 Structural Models: The Merton Framework (Preview)

### 35.6.1 Equity as a Call Option on Firm Value

Before the reduced-form models used for CDS pricing, structural models provided the first rigorous framework for credit risk. O'Kane explains: "The history of credit modelling began with the work of Merton (1974), in which he proposed an option-based model for corporate default. Merton's idea was to model default as the event which occurs when the value of the assets of a firm fall below the value of the firm's debt."

The Merton model assumes a simplified capital structure:
- Face value $F$ of zero-coupon debt maturing at time $T$
- Equity $E$ that pays no dividends
- Total firm value $A(t) = D(t) + E(t)$

At maturity, two states are possible:

**Solvency** ($A(T) \geq F$): Bondholders receive $F$; equity holders receive $A(T) - F$

**Insolvency** ($A(T) < F$): Bondholders receive $A(T)$; equity holders receive nothing

The equity payoff is therefore:

$$E(T) = \max[A(T) - F, 0]$$

This is the payoff of a **call option** on firm value struck at the debt face value.

The debt payoff can be written as:

$$D(T) = F - \max[F - A(T), 0] = \min[F, A(T)]$$

Bondholders are effectively long cash and short a put option on firm value.

### 35.6.2 Distance to Default

Under the assumption that firm value follows geometric Brownian motion with volatility $\sigma_A$, Merton derived closed-form expressions for equity and debt values using Black-Scholes:

$$E(t) = A(t)\Phi(d_1) - Fe^{-r(T-t)}\Phi(d_2)$$

where

$$d_1 = \frac{\ln(A(t)/F) + (r + \frac{1}{2}\sigma_A^2)(T-t)}{\sigma_A\sqrt{T-t}}, \quad d_2 = d_1 - \sigma_A\sqrt{T-t}$$

The survival probability (probability firm value exceeds debt at maturity) is:

$$Q(t,T) = \Pr(A(T) \geq F) = \Phi(d_2)$$

The term $d_2$ can be interpreted as the "distance to default" in standardized units—how many standard deviations the firm's assets lie above the default threshold.

### 35.6.3 Endogenous Recovery in the Merton Model

One elegant feature of the Merton model is that it determines recovery endogenously. O'Kane notes: "the Merton model not only allows us to price equity and bonds, it also allows us to determine the recovery rate endogenously."

The recovery rate depends on how far asset value has fallen below the debt threshold at default—it is not a fixed input but emerges from the model.

### 35.6.4 Limitations of Structural Models for Pricing

O'Kane lists key limitations:

1. **Simplified capital structure.** Real firms have multiple debt tranches with different seniorities and maturities. "Merton's model assumes that the bonds are zero coupon. It is possible to extend Merton's model to handle coupon paying bonds as in Geske (1977). However, the computational complexity of the pricing formulae increases significantly—a coupon paying bond is a compound option, an option on an option in this framework."

2. **Default only at maturity.** The basic model doesn't allow default before $T$.

3. **Asset value transparency.** Firm asset values are not directly observable.

4. **Short-term spreads vanish.** "The credit spread for firms for which $A(t) > F$ always tends to zero as $T-t \rightarrow 0$. This is not consistent with the credit markets in which even corporates with very high credit ratings have a finite spread at very short maturities."

Despite these limitations, O'Kane concludes that "structural models perform best as a tool for augmenting the traditional balance sheet analysis methods of credit analysts." The Moody's KMV model is a prominent commercial application.

For derivatives pricing, the market uses **reduced-form models** (Chapter 36) that directly model hazard rates and calibrate to market spreads.

---

## 35.7 Worked Examples

### Example A: Recovery Rate from Post-Default Bond Price

A bond trades at $P = 28$ per 100 par shortly after default.

**Recovery rate:**
$$R = \frac{P}{100} = \frac{28}{100} = 0.28$$

**Loss given default:**
$$\text{LGD} = 1 - R = 1 - 0.28 = 0.72$$

**Interpretation:** For every $100 of notional, the protection seller pays $72.

---

### Example B: CDS Protection Leg Payoff

CDS notional $N = \$10{,}000{,}000$. Recovery rates of 20%, 40%, and 60%.

| Recovery $R$ | LGD $= 1-R$ | Protection Payment $N \times \text{LGD}$ |
|--------------|-------------|------------------------------------------|
| 0.20 | 0.80 | $\$8{,}000{,}000$ |
| 0.40 | 0.60 | $\$6{,}000{,}000$ |
| 0.60 | 0.40 | $\$4{,}000{,}000$ |

**Unit check:** Dollars × dimensionless = dollars. ✓

**Intuition:** Recovery rate directly scales the payout. Every 10 percentage points of recovery reduces the payout by $1,000,000.

---

### Example C: Lehman Brothers Recovery

Hull reports that the Lehman Brothers CDS auction in 2008 produced a cash payout of 91.375% of principal, implying a recovery of approximately 8.625 cents on the dollar.

**Parameters:**
- Cash payout to protection buyers: 91.375% of principal
- Implied recovery: $R = 1 - 0.91375 = 0.08625 \approx 8.6\%$
- Notional $N = \$10{,}000{,}000$

**CDS payoff:**
$$\Pi = N(1 - R) = 10{,}000{,}000 \times 0.91375 = \$9{,}137{,}500$$

**Context:** Hull notes that "when Lehman defaulted in September 2008, there was about $400 billion of CDS contracts and $155 billion of Lehman debt outstanding." The extremely low recovery—among the lowest in major corporate defaults—meant CDS protection buyers received nearly the full notional.

---

### Example D: Delivery Option Value After Restructuring

Following a restructuring credit event, two deliverable bonds trade at different prices:
- Bond A: $43 per 100
- Bond B: $37 per 100

CDS notional $N = \$10{,}000{,}000$.

**Without delivery option** (deliver Bond A):
$$\Pi = N \times \frac{100 - 43}{100} = 10{,}000{,}000 \times 0.57 = \$5{,}700{,}000$$

**With delivery option** (deliver cheapest, Bond B):
$$\Pi = N \times \frac{100 - 37}{100} = 10{,}000{,}000 \times 0.63 = \$6{,}300{,}000$$

**Delivery option value:**
$$\$6{,}300{,}000 - \$5{,}700{,}000 = \$600{,}000$$

This is 6 points per 100 notional, matching O'Kane's example.

---

### Example E: Portfolio Jump-to-Default Exposure

A portfolio has four CDS positions where you are short protection (you pay on credit event):

| Name | Notional |
|------|----------|
| 1 | $5mm |
| 2 | $5mm |
| 3 | $10mm |
| 4 | $20mm |
| **Total** | $40mm |

**Case 1:** Uniform recovery $R = 40\%$ → LGD = 60%

| Name | Exposure $= N \times \text{LGD}$ |
|------|----------------------------------|
| 1 | $5 \times 0.60 = \$3.0mm$ |
| 2 | $5 \times 0.60 = \$3.0mm$ |
| 3 | $10 \times 0.60 = \$6.0mm$ |
| 4 | $20 \times 0.60 = \$12.0mm$ |
| **Total** | $\$24.0mm$ |

**Case 2:** Mixed recoveries $R_1 = 20\%, R_2 = 40\%, R_3 = 50\%, R_4 = 60\%$

| Name | $R$ | LGD | Exposure |
|------|-----|-----|----------|
| 1 | 0.20 | 0.80 | $\$4.0mm$ |
| 2 | 0.40 | 0.60 | $\$3.0mm$ |
| 3 | 0.50 | 0.50 | $\$5.0mm$ |
| 4 | 0.60 | 0.40 | $\$8.0mm$ |
| **Total** | | | $\$20.0mm$ |

**Risk management insight:** Exposure is bounded by total notional but depends critically on recovery assumptions.

---

### Example F: Gray Zone P&L Risk

A trader holds a $\$10mm$ bond position hedged with $\$10mm$ notional CDS protection. The bond collapses from 85 to 35, but no credit event has been declared (grace period ongoing).

**Bond P&L:**
$$\Delta P_{\text{bond}} = 10{,}000{,}000 \times \frac{35 - 85}{100} = -\$5{,}000{,}000$$

**CDS P&L (mark-to-market, no trigger yet):**
Assume CDS spread widened from 300bp to 2500bp, and 5Y risky duration ≈ 3.5 years.

$$\Delta P_{\text{CDS}} \approx 10{,}000{,}000 \times (0.25 - 0.03) \times 3.5 = +\$7{,}700{,}000$$

**Net P&L:** Approximately +$2.7mm in mark-to-market terms.

**The risk:** If the company cures the payment and avoids credit event:
- Bond may recover to 60–70
- CDS spread collapses, protection loses value
- The hedge "worked" but timing created P&L volatility

> **Desk Reality:** During this gray zone, the trader may face:
> - Margin calls on the bond (funded at distressed price)
> - Collateral posting on CDS (receiving margin as spread widens)
> - Accounting mismatch if bond is held at cost and CDS is marked-to-market
> - Uncertainty about whether to unwind or hold through potential credit event

---

### Example G: Coverage Ratio Calculation

A Baa-rated corporate has:
- 7-year historical default rate: 4.7% cumulative → annual intensity ≈ 0.7%
- Market CDS spread: 225 bp (5-year)
- Assumed recovery: 40%

**Actuarial spread:**
$$S_{\text{actuarial}} = \lambda_{\text{historical}} \times (1 - R) = 0.007 \times 0.60 = 0.42\% = 42 \text{ bp}$$

**Coverage ratio:**
$$\text{CR} = \frac{S_{\text{market}}}{S_{\text{actuarial}}} = \frac{225}{42} = 5.4\times$$

**Interpretation:** The market demands 5.4× the actuarial expected loss to hold this credit risk. The 183 bp excess spread compensates for:
- Systematic default risk (defaults cluster in recessions)
- Recovery uncertainty (could be much lower than 40%)
- Liquidity premium
- Model uncertainty

---

### Example H: Portfolio Jump-to-Default with Correlation Stress

Extending Example E, consider the impact of the default-recovery correlation during a stressed period.

**Base case:** $R = 40\%$ for all names, total JTD exposure = $24mm (from Example E)

**Stressed case:** In a recession with elevated defaults, assume recovery drops to 25%:
$$\text{Total JTD} = 40 \times (1 - 0.25) = \$30mm$$

**Increase in exposure:**
$$\Delta \text{Exposure} = 30 - 24 = \$6mm \text{ (25\% increase)}$$

This illustrates the "doubly bad" phenomenon: when you most need diversification (high default environment), recovery rates fall, amplifying losses.

---

## 35.8 Practical Notes

### Common Pitfalls

1. **Treating recovery as fixed.** Recovery distributions are broad (σ ≈ 25%). A single assumed value masks significant uncertainty.

2. **Confusing bond price with ultimate recovery.** The credit derivatives settlement price can differ from workout recovery—sometimes significantly.

3. **Ignoring the delivery option.** For restructuring events, cheapest-to-deliver dynamics can significantly affect CDS payoffs.

4. **Forgetting accrued premium on default.** It's part of the standard CDS cashflows and affects hedge accounting.

5. **Assuming universal credit event definitions.** Restructuring clauses differ by region and contract vintage.

6. **Using historical default rates for pricing.** Hull shows that risk-neutral hazard rates implied from spreads are typically 2–10× historical rates. The difference is a risk premium, not a market inefficiency.

7. **Ignoring the gray zone.** The period between economic distress and contractual credit event creates basis risk, funding risk, and accounting mismatches.

8. **Assuming APR holds strictly.** Absolute priority violations are common in practice, affecting recovery rate analysis by seniority.

### Sanity Checks

| Check | Condition |
|-------|-----------|
| Payout bounds | $0 \leq \Pi_{\text{prot}} \leq N$ |
| Recovery bounds | $0 \leq R \leq 1$ |
| Settlement consistency | If $R = P/100$, physical and cash settlement align economically |
| LGD positivity | $\text{LGD} = 1 - R \geq 0$ |
| Accrued premium | $N \times S \times \alpha \geq 0$ for $S, \alpha \geq 0$ |
| Coverage ratio | CR > 1 for most credits (market demands risk premium) |
| Seniority ordering | Senior recovery > Subordinated recovery (on average) |

### Implementation Notes

- **Date handling:** Payment schedule generation and accrual fractions require careful calendar logic
- **Notional and sign conventions:** Buyer receives protection payment; seller pays it
- **Settlement timing:** Physical settlement can take up to 72 calendar days; model accordingly
- **Short squeeze risk:** When CDS notional exceeds deliverable supply, prices can be distorted
- **Upfront conversion:** For distressed names, convert between running spread and upfront using risky duration

---

## Summary

1. **Economic default** (distress visible in prices) is not the same as a **contractual credit event** (the legal trigger for CDS protection).

2. **Par vs distressed trading** represents a fundamental shift in how markets quote instruments—from yield/spread focus to recovery/price focus.

3. **Recovery rate** $R$ is the bond price shortly after default as a fraction of face value. The **loss given default** is $\text{LGD} = 1 - R$.

4. Historical recovery averages approximately 35% for senior unsecured bonds (mean 34.89% per O'Kane), but with high variability (σ ≈ 25–27%). A common assumption is 40% (Hull).

5. Recovery rates are **negatively correlated** with default rates—a bad default year is doubly bad. The relationship has $R^2 \approx 0.51$ historically.

6. **Absolute Priority Rule** violations are common in practice, meaning subordinated recovery can exceed strict waterfall predictions.

7. **Credit risk premium** causes market spreads to exceed actuarial expectations by 2–10× (coverage ratio), compensating for systematic risk, recovery uncertainty, and liquidity.

8. **Credit events** include hard events (bankruptcy, failure to pay) and the soft event (restructuring).

9. **ISDA Determinations Committees** govern credit event adjudication since 2009, with the **auction mechanism** determining recovery prices.

10. **Restructuring clauses** (Old-Re, Mod-Re, Mod-Mod-Re, No-Re) create regional differences in CDS contracts and affect spread levels.

11. The CDS **protection leg payoff** is $N(1-R)$ in cash settlement.

12. The **delivery option** gives protection buyers the right to choose the cheapest deliverable—valuable after soft events.

13. The **Merton model** provides structural intuition (equity as call option on firm value) but is not used directly for CDS pricing due to limitations with short-term spreads and complex capital structures.

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| Credit event | Contractual trigger for CDS protection payment | Determines when protection pays—not the same as economic distress |
| Gray zone | Period between economic distress and credit event | Creates basis risk, funding risk, accounting mismatch |
| Recovery rate $R$ | Bond price shortly after default ÷ face value | Scales the protection payment; varies significantly (σ ≈ 25%) |
| Loss given default | $1 - R$ | The fraction of notional paid on protection leg |
| Coverage ratio | Market spread / Actuarial spread | Measures risk premium; typically 2–10× for investment grade |
| Hard credit event | Causes all debt to become immediately due | Single distressed price for all obligations |
| Soft credit event | Restructuring only | Debt trades with term structure of prices |
| ISDA DC | Determinations Committee | Governs credit event adjudication since 2009 |
| ISDA auction | Two-stage price discovery process | Determines recovery for cash settlement |
| Delivery option | Right to choose which deliverable to surrender | Creates value, especially after restructuring |
| Physical settlement | Deliver obligations for par | Protection buyer can exploit cheapest-to-deliver |
| Cash settlement | Receive par minus auction price | Standard for indices; avoids delivery logistics |
| APR violation | Senior paid less than full before junior receives | Common in practice; affects recovery analysis |
| Merton model | Equity = call option on firm value | Structural intuition for credit risk |
| Distance to default | How far firm value is above debt threshold | Credit risk metric from structural models |
| Risk-neutral vs real-world | Implied from spreads vs historical statistics | Use risk-neutral for pricing, real-world for scenario analysis |

---

## Notation Summary

| Symbol | Definition |
|--------|------------|
| $N$ | CDS notional |
| $R$ | Recovery rate (fraction of par) |
| $\text{LGD}$ | Loss given default $= 1 - R$ |
| $P$ | Bond price per 100 par |
| $S$ | CDS spread (annual rate, decimal) |
| $\lambda$ | Hazard rate (instantaneous default intensity) |
| $\alpha$ | Day-count fraction (typically ACT/360) |
| $\Pi_{\text{prot}}$ | Protection leg payment |
| $\text{CR}$ | Coverage ratio (market spread / actuarial spread) |
| $A(t)$ | Firm asset value (Merton model) |
| $F$ | Debt face value (Merton model) |
| $E(t)$ | Equity value (Merton model) |
| $\sigma_A$ | Asset volatility (Merton model) |
| $d_1, d_2$ | Black-Scholes parameters (Merton model) |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is a "credit event"? | The legal/contractual trigger for CDS protection payment—similar to but not identical to default |
| 2 | What is the "gray zone" in credit? | Period between economic distress and contractual credit event; creates basis and funding risk |
| 3 | Hard vs soft credit event—key difference? | Hard: all debt trades at single price. Soft (restructuring): term structure of prices persists |
| 4 | Name the only soft credit event | Restructuring |
| 5 | Define recovery rate $R$ | Bond price shortly after default divided by face value |
| 6 | Define LGD | Loss given default $= 1 - R$ |
| 7 | CDS protection leg payoff formula | $\Pi = N(1 - R)$ |
| 8 | What is physical settlement? | Buyer delivers obligations; receives par in cash |
| 9 | What is cash settlement? | Seller pays par minus recovery price (determined by auction) |
| 10 | What is the delivery option? | Protection buyer's right to choose which deliverable to surrender |
| 11 | When is the delivery option most valuable? | After soft credit events (restructuring) with heterogeneous bond prices |
| 12 | What happens to premium leg after credit event? | Stops, but accrued premium since last payment date is paid |
| 13 | Day count for CDS accrual? | Typically Actual/360 |
| 14 | Mean recovery for senior unsecured bonds? | Approximately 35% (34.89% per O'Kane); 40% commonly assumed |
| 15 | Relationship between recovery and default rates? | Negative correlation—high default years have low recoveries ($R^2 \approx 0.51$) |
| 16 | What is the coverage ratio? | Market spread / Actuarial spread; measures risk premium |
| 17 | Typical coverage ratio for investment grade? | 5–15× (market demands large premium over expected loss) |
| 18 | What does Mod-Re mean? | Modified-Restructuring clause—limits deliverable maturities (US standard) |
| 19 | What does No-Re mean? | No-Restructuring—removes restructuring as credit event |
| 20 | What is the ISDA Determinations Committee? | Regional panel that adjudicates credit event determinations since 2009 |
| 21 | How does the ISDA auction work? | Two stages: (1) dealers set IMM, (2) open auction with limit orders determines final price |
| 22 | Merton model: equity is equivalent to? | Call option on firm value struck at debt face value |
| 23 | Merton model: bondholders are equivalent to? | Long cash, short put on firm value |
| 24 | What is "distance to default"? | How many standard deviations firm value exceeds debt threshold |
| 25 | Why aren't structural models used for CDS pricing? | Limitations: simplified capital structure, default only at maturity, short-term spreads vanish |
| 26 | What is an APR violation? | Senior creditors receive less than full recovery while junior creditors receive something |
| 27 | Par trading vs distressed trading focus? | Par: yield/spread. Distressed: recovery/price ("cents on the dollar") |
| 28 | When does CDS shift to upfront quoting? | When spreads exceed ~1000bp; protection quoted as % of notional upfront |

---

## Mini Problem Set

**1.** A CDS has notional $\$25mm$ and recovery $R = 0.35$. Compute the protection payment.

**2.** Same as (1) but recovery is quoted as price 42 per 100. Compute $R$ and the protection payment.

**3.** A bond trades at 28 per 100 shortly after default. Compute market-implied recovery and LGD.

**4.** Premium accrual: Notional $\$50mm$, spread 150 bp, 40 days since last premium date on ACT/360. Compute accrued premium.

**5.** Show algebraically that physical and cash settlement produce the same economic result when the settlement price equals the deliverable market price.

**6.** Delivery option: Two deliverables at 45 and 38 after restructuring, notional $\$10mm$. Compute the delivery option value.

**7.** Explain why restructuring is classified as "soft" and how that impacts post-event pricing.

**8.** A trader hedges a $\$20mm$ bond position with CDS. The bond price collapses from 80 to 40 but no credit event is triggered for 30 days. Describe the hedge P&L risks qualitatively.

**9.** Given recovery uncertainty with three scenarios ($R = 0.20$, prob 50%; $R = 0.40$, prob 30%; $R = 0.60$, prob 20%) and notional $\$10mm$, compute expected protection payment.

**10.** In the Merton model, if firm value $A = \$120mm$, debt face value $F = \$100mm$, and $d_2 = 1.0$, compute the survival probability.

**11.** Why are risk-neutral hazard rates higher than historical default rates? Give two reasons.

**12.** A portfolio has $100mm total notional across 20 names. If the average LGD is 60%, what is the maximum single-name jump-to-default loss?

**13.** A BBB-rated credit has historical 5-year default rate of 2.2%, assumed recovery of 40%, and market CDS spread of 180bp. Calculate the coverage ratio and interpret.

---

## Solution Sketches (Questions 1–6, 13)

**1.** $\Pi = N(1 - R) = 25 \times (1 - 0.35) = \$16.25mm$

**2.** $R = 0.42$. $\Pi = 25 \times (1 - 0.42) = 25 \times 0.58 = \$14.5mm$

**3.** $R = 0.28$, LGD $= 0.72$

**4.** $\alpha = 40/360 = 0.1111$. Accrued $= 50{,}000{,}000 \times 0.015 \times 0.1111 = \$83{,}333$

**5.** Cash: $N(1 - P/100)$. Physical: Buy at $P$, deliver for 100 → gain $= N(100 - P)/100 = N(1 - P/100)$. QED.

**6.** Per 100: $45 - 38 = 7$ points. For $\$10mm$: $0.07 \times 10{,}000{,}000 = \$700{,}000$

**13.** Annual default intensity ≈ $0.022/5 = 0.44\%$. Actuarial spread = $0.0044 \times 0.60 = 0.264\% = 26.4$ bp. Coverage ratio = $180/26.4 = 6.8\times$. Interpretation: Market demands nearly 7× the actuarial expected loss, reflecting systematic risk premium, recovery uncertainty, and liquidity premium typical of investment-grade credit.

---

## References

- O'Kane, *Modelling Single-name and Multi-name Credit Derivatives* (credit events, recovery concepts, default definitions and historical context)
- Hull, *Options, Futures, and Other Derivatives* (credit risk, CDS basics, recovery and hazard-rate intuition)
- Altman et al. on empirical recovery statistics and recovery–default correlation (as cited in O'Kane)
