# Chapter 45: CDS Indices — Structure, Quoting, and Lifecycle

---

## Introduction

"CDX.NA.IG.42 5Y" — what does each part of this cryptic string mean, and why does misunderstanding it matter?

When a trader says they're "buying CDX," they're entering a single contract that provides credit exposure to 125 investment-grade North American companies simultaneously. This is the power of CDS indices: one trade to express a macro credit view, one instrument to hedge a diverse credit book, and — crucially — far more liquidity than trading 125 single-name CDS contracts individually. O'Kane emphasizes that "CDS portfolio indices have transformed the credit derivatives markets by providing a highly liquid product which investors can use to assume or hedge the risk of a broad portfolio of credit exposures." But this convenience comes with complexity. The naming convention encodes geography, credit quality, vintage, and tenor. The quoting convention shifts between spread and price depending on the index family. And the lifecycle — from new series launch through constituent defaults to roll dates — creates operational landmines for the unwary.

**Why this matters:** Consider a trader who books a CDX.NA.HY trade using the wrong quoting convention. High-yield indices quote as "bond price" (not spread), so confusion between the two conventions can cause upfront payment errors of several percent of notional — on a $100 million trade, that's millions of dollars. Similarly, failing to track which series is "on-the-run" versus "off-the-run" affects liquidity, hedge effectiveness, and roll P&L. These aren't theoretical concerns; they're daily operational realities on credit trading desks.

> **Why Middle Office Cares**
>
> Your daily risk report shows CDX.IG DV01 exposure aggregated across the desk. When the index rolls from Series 41 to Series 42, positions need rebooking from the old series to the new. A missed roll booking causes P&L breaks when the old series stops updating and reconciliation issues when positions appear on "dead" instruments. Understanding the roll calendar, the index factor after defaults, and the quoting convention is essential for anyone producing or consuming credit risk reports.

**Chapter roadmap:** This chapter covers the **market structure and operational mechanics** of CDS indices:

- **Section 45.1** decodes index naming conventions — the systematic structure behind identifiers like "iTraxx Europe Series 6"
- **Section 45.2** explains quoting conventions — when indices quote as spread versus price, and how to convert between them
- **Section 45.3** covers the index lifecycle — from series launch through rolls to maturity
- **Section 45.4** addresses default handling within indices — the timeline from credit event to auction to settlement
- **Section 45.5** previews the intrinsic spread concept and index basis (detailed treatment in Chapter 46)
- **Section 45.6** presents worked examples for common operational calculations

**Relationship to other chapters:** Chapter 41 covers the *economics* of CDS indices — coupon mechanics, premium leg valuation, and the upfront-spread relationship. This chapter focuses on *market structure and operations*. Chapter 46 addresses the intrinsic spread calculation and index basis in depth. Chapter 40 provides the detailed CDS auction mechanics that we reference for default settlement.

---

## Notation for This Chapter

| Symbol | Definition |
|--------|------------|
| $N$ | Index trade notional (USD) |
| $M$ | Number of index constituents at inception (e.g., 125 for CDX.NA.IG) |
| $D$ | Number of constituents that have defaulted |
| $f$ | Index factor: $f = (M - D) / M$ |
| $C$ or $c$ | Annualized running coupon rate in decimal (e.g., 100 bp $= 0.01$) |
| $S_I(t,T)$ | Market-quoted flat index spread at time $t$ for maturity $T$ |
| $U_I$ | Upfront as percent of notional (positive = paid by protection buyer; negative = received) |
| $\text{RPV01}_I$ | Risky PV01 of the index (PV of 1 bp running premium) |
| $P_{\text{bond}}$ | Bond-price quote (percent of notional); $U_I = 100 - P_{\text{bond}}$ |
| $FP$ | Auction final price (value per 100 of deliverable) used for cash settlement |
| $S_I^{\text{intr}}$ | Intrinsic spread (RPV01-weighted average of constituent spreads) |

---

## 45.1 Index Naming Conventions: Decoding the Identifier

A CDS index identifier encodes multiple pieces of information in a structured format. Understanding this structure is essential for correct trade booking and risk management.

### 45.1.1 The Standard Index Families

O'Kane provides a comprehensive taxonomy of the main CDS indices, noting that "the arrival of the first CDS portfolio indices in 2002 heralded the start of a new era for the credit derivatives market." Table 45.1 summarizes the key index families.

**Table 45.1: Major CDS Index Families**

| Index Name | Geography | Credit Quality | Number of Credits |
|------------|-----------|----------------|-------------------|
| CDX.NA.IG | North America | Investment Grade | 125 |
| CDX.NA.HY | North America | High Yield | 100 |
| CDX.NA.XO | North America | Crossover | 35 |
| CDX.EM | Emerging Markets | Sovereign | 15 |
| CDX.EM.Diversified | Emerging Markets | Sovereign + Corporate | 40 |
| iTraxx Europe | Europe | Investment Grade | 125 |
| iTraxx Europe HiVol | Europe | High Volatility | 30 |
| iTraxx Europe Crossover | Europe | Crossover | 40-50 |
| iTraxx Japan | Japan | Investment Grade | 50 |
| iTraxx Asia ex-Japan | Asia ex-Japan | Investment Grade | 50 |
| iTraxx Australia | Australia | Investment Grade | 25 |

*Source: O'Kane Ch. 10, Table 10.1*

> **Note on constituent counts:** The numbers shown are standards at launch. O'Kane clarifies that "the number of credits in any one index may be lower than the number shown here due to their removal following credit events" — defaulted names are removed without replacement within a series.

> **What is a "crossover" credit?** O'Kane defines crossover as "a credit which was previously investment grade but has since been downgraded."

> **Desk Reality: Screen Conventions**
>
> On Bloomberg terminals, you'll see tickers like:
> - **CDX IG CDSI** — CDX.NA.IG generic (rolls to current on-the-run)
> - **ITRX EUR CDSI** — iTraxx Europe generic
> - **CDX HY CDSI** — CDX.NA.HY generic
>
> For specific series: **CDX IG S42** refers to Series 42. Markit's RED (Reference Entity Database) codes are used for official constituent identification in trade confirmations. When booking, always verify the RED code matches the intended series.

### 45.1.2 Index Governance and Administration

O'Kane explains that "since there are no published estimates for the trading volume of different issuers in the CDS market, determining which issuers are the most liquid is not easy." To ensure unbiased composition decisions, "the decision-making process is managed by independent index administrators in consultation with a consortium of investment banks, based on defined and published rules."

The owner and administrator of the CDX and iTraxx indices is **Markit Group Limited** (now part of IHS Markit/S&P Global). This matters operationally because Markit publishes official constituent lists, coupon fixings, and settlement prices that serve as reference data for trade processing.

### 45.1.3 Anatomy of an Index Identifier

A complete index identifier follows the pattern:

$$\boxed{\text{[Family].[Region].[Quality].[Series]}\ [\text{Tenor}]}$$

**Example: CDX.NA.IG.42 5Y**

| Component | Value | Meaning |
|-----------|-------|---------|
| CDX | — | Index family (North American indices) |
| NA | — | North America |
| IG | — | Investment Grade |
| 42 | — | Series number (42nd vintage) |
| 5Y | — | 5-year maturity contract |

**Example: iTraxx Europe Series 6 7Y**

| Component | Value | Meaning |
|-----------|-------|---------|
| iTraxx | — | Index family (European indices) |
| Europe | — | European constituents |
| (IG implied) | — | Investment grade unless otherwise noted |
| Series 6 | — | 6th vintage |
| 7Y | — | 7-year maturity contract |

### 45.1.4 Why Indices Exist: Operational Advantages

O'Kane enumerates several advantages of portfolio CDS indices that explain their market success:

1. **Diversified exposure in one trade:** "Since the indices contain as many as 125 credits, the portfolio CDS index allows investors to take a broad exposure to the credit markets in a single transaction."

2. **Efficient shorting:** "The portfolio CDS index allows hedgers to go short the credit markets in a single transaction. This is useful for credit funds, hedge funds or banks who wish to hedge a broad exposure to the credit markets."

3. **Capital efficiency:** "Since the portfolio CDS indices have a small initial cost, they can be used to take a leveraged credit exposure. In addition, it makes them efficient for those who fund above Libor."

4. **Derivative underlyings:** "Portfolio CDS indices can be used as the underlying for other derivative products such as synthetic loss tranches and spread options." This is the foundation for the tranche products covered in Chapters 48-51.

5. **Sector trading:** "By allowing the trading of constituent subindices, it becomes possible to take relative value views on sectors." Both CDX and iTraxx publish sector subindices that reference subsets of the main index.

**Exceptional liquidity:** O'Kane notes that "the format of these indices was driven primarily by the need to make them as easy to trade as possible and by doing so, to encourage liquidity. This has succeeded. Liquidity of this market is now exceptional, with **bid-offer spreads of one quarter of a basis point** on the most liquid investment grade indices rising to just **4 bp on the high yield index**."

> **Desk Reality: Liquidity Tiers**
>
> In practice, credit traders rank index liquidity roughly as:
>
> 1. **CDX.NA.IG 5Y** — Most liquid credit product globally; the "SPX of credit"
> 2. **iTraxx Europe Main 5Y** — European benchmark; comparable liquidity
> 3. **CDX.NA.HY** — Deep market but wider spreads than IG
> 4. **iTraxx Crossover** — European high-yield proxy
> 5. **Off-the-run series** — Less liquid; bid-offer can widen materially
>
> Execution note: on-the-run indices can absorb very large risk quickly; off-the-run series can be markedly harder to trade, especially in stressed markets.

> **Tradable vs. benchmark indices:** O'Kane distinguishes CDS portfolio indices from benchmark indices: "While the CDS indices are designed with the primary aim of tradability, benchmark indices are more concerned with providing an accurate reflection of the performance of the 'market'."

### 45.1.5 Series and Version: The Vintage System

O'Kane explains that "most CDS indices are issued or rolled semi-annually with a coupon which is fixed for the lifetime of the issue." Each issuance creates a new **series** with a fixed constituent list (except for default removals).

**Series numbering:**
- The series number increments each roll for a given index family.
- The exact "current" series depends on the date; always book the series specified on the confirmation (or the official index documentation).

**On-the-run vs. off-the-run:**
- **On-the-run:** The most recently issued series — "from the time a new index is issued to the time that the next series of the index is issued, this index is known as the on-the-run index" (O'Kane)
- **Off-the-run:** All previous series remain tradable but with reduced liquidity

> **Practical note:** Liquidity concentrates heavily in the on-the-run series. O'Kane notes that for index options, "expiries beyond six months are rare because liquidity drops immediately after the next roll," reflecting the dominance of on-the-run trading.

### 45.1.6 Maturity Labels and Actual Tenors

O'Kane emphasizes an important detail: "the so-called 3Y, 5Y, 7Y or 10Y on-the-run indices do not have exactly 3, 5, 7 or 10 years to maturity on their issue date."

**The ±3 month rule:**
- At issuance, a "T-year" index typically has $T + 3$ months to maturity
- By the next roll (6 months later), it has $T - 3$ months to maturity
- Example: A 5Y index issued in March 2006 maturing in June 2011 has 63 months at issuance

**Maturity date conventions:**
- Indices issued on March 20 mature on June 20 of the appropriate year
- Indices issued on September 20 mature on December 20 of the appropriate year

**Table 45.2: Standard Maturity Points**

| Maturity Label | Most Liquid? | Typical Use |
|----------------|--------------|-------------|
| 3Y | No | Short-dated hedging |
| 5Y | Yes | Benchmark trading, hedging |
| 7Y | No | Curve trades |
| 10Y | Yes | Long-dated exposure |

*Source: O'Kane Ch. 10*

O'Kane confirms: "In the early days of the CDS index, the only tenor was five years. However, there is now a liquid index with a tenor of 10 years and indices with lower liquidity at three and seven years."

---

## 45.2 Quoting Conventions: Spread vs. Price

One of the most operationally critical aspects of CDS indices is understanding when they quote as **spread** versus when they quote as **price**. Getting this wrong leads to booking errors.

### 45.2.1 The Two Quoting Regimes

O'Kane describes the bifurcation clearly: "Most investors prefer to trade the indices in unfunded format. For investment grade contracts such as the CDX IG and the iTraxx Europe, the market quotes an **index spread** which represents the flat spread at which the index should be priced if it were a simple CDS contract."

However, "other lower credit quality indices such as the EM and HY indices are quoted using the **bond price convention** ... in which the upfront value is computed simply by subtracting 100."

**Table 45.3: Quoting Convention by Index Family (Coupon is Series-Specific)**

| Index Family | Typical Quoting Convention | Fixed Coupon | Reason |
|--------------|---------------------------|--------------|--------|
| CDX.NA.IG | Spread | Fixed at series inception | PV01 conversion is manageable |
| iTraxx Europe | Spread | Fixed at series inception | PV01 conversion is manageable |
| CDX.NA.HY | Bond Price | Fixed at series inception | Avoid PV01 disputes by quoting price |
| CDX.EM | Bond Price | Fixed at series inception | Avoid PV01 disputes by quoting price |
| iTraxx Crossover | Bond Price | Fixed at series inception | Avoid PV01 disputes by quoting price |

> **Desk Reality: "Points Upfront" Language**
>
> When traders say "CDX is 50 over," they mean the current spread is 50 bp above the fixed coupon (e.g., if coupon is 100 bp, current spread is 150 bp). This implies the protection buyer pays upfront (index trades below par).
>
> Conversely, "CDX is 20 under" means the spread is 20 bp below the coupon (e.g., if coupon is 100 bp, spread is 80 bp). The protection buyer receives upfront (index trades above par).
>
> For HY indices with a high fixed coupon (e.g., 500 bp), "HY is 200 over" means the spread is 700 bp. The larger coupon means upfronts are larger in dollar terms.

### 45.2.2 Spread Quoting (Investment Grade)

When an index quotes by spread, the **upfront payment** is derived from the difference between the quoted spread and the contractual coupon, scaled by the risky PV01:

$$\boxed{U_I = (S_I - C) \times \text{RPV01}_I}$$

**Interpretation:**
- If $S_I > C$: Spread has widened since issuance → protection buyer **pays** upfront (index "trades below par")
- If $S_I < C$: Spread has tightened → protection buyer **receives** upfront (index "trades above par")

**The flat curve convention:** O'Kane explains that "market convention dictates that the index curve used to value an index has a flat term structure. So if we wish to value the 7Y CDX NA Investment Grade index, we would simply use a flat CDS curve at a level of 44 bp." This is important: the $\text{RPV01}_I$ in the formula above is computed using a flat spread assumption, not a term structure.

**Example from O'Kane (Table 10.2, November 2006):**

| Index | Series | Coupon (bp) | Index Spread (bp) | Implied Bond Price |
|-------|--------|-------------|-------------------|-------------------|
| CDX.NA.IG | 7 | 40 | 34 | 100.27% |
| iTraxx Europe | 6 | 30 | 24 | 100.28% |

The CDX.NA.IG spread (34 bp) is below the coupon (40 bp), meaning credit quality has improved since issuance. A protection buyer entering receives upfront (reflected in the bond price above 100).

### 45.2.3 Bond Price Quoting (High Yield / EM)

O'Kane explains the rationale: "this convention has the advantage that it avoids any disagreement about the value of the index PV01 which is required to convert a spread-based quotation into an upfront value."

**The conversion is simple:**

$$\boxed{U_{\%} = 100\% - P_{\text{bond}}}$$

**Examples:**
- Bond price = 102.18% → Receive 2.18% upfront (so $U_{\%} = -2.18\%$)
- Bond price = 99.50% → Pay 0.50% upfront (so $U_{\%} = +0.50\%$)

**Dollar upfront from bond price:**

$$\boxed{U_{\$} = N \times (1 - P_{\text{bond}}/100)}$$

**Why bond price avoids disputes:**

The spread-to-upfront conversion requires computing $\text{RPV01}_I$, which depends on assumptions about the flat spread level used for discounting. O'Kane notes that the index RPV01 "depends implicitly on $S_I$" — solving for the spread that matches the upfront may require iteration. By quoting directly in price terms, counterparties avoid disagreements about the correct PV01.

### 45.2.4 Converting Between Spread and Price

**From spread to upfront (when spread-quoted):**

Given spread $S_I$, coupon $C$, and $\text{RPV01}_I$:

$$U_{\%} = (S_I - C) \times \text{RPV01}_I$$

**From upfront to implied spread:**

$$S_I = C + \frac{U_{\%}}{\text{RPV01}_I}$$

**Hull's example (Example 25.1):** For iTraxx Europe:
- Quoted spread: 34.5 bp (actual/360) ≈ 34 bp (actual/actual)
- Coupon: 40.6 bp (actual/actual)
- RPV01 ("duration"): 4.447 years
- Price calculation: $100 - 100 \times 4.447 \times (0.00345 - 0.00406) = 100.27$

This confirms: spread below coupon means index trades above par.

### 45.2.5 The Index Factor

When defaults occur, the index notional reduces. The **index factor** tracks this:

$$\boxed{f = \frac{M - D}{M}}$$

where $M$ is the original constituent count and $D$ is the number of defaulted names.

**Examples:**
- CDX.IG with 0 defaults: $f = 125/125 = 1.000$
- CDX.IG with 2 defaults: $f = 123/125 = 0.984$
- CDX.HY with 5 defaults: $f = 95/100 = 0.950$

**Outstanding notional:**

$$\boxed{N_{\text{out}} = N \times f}$$

> **Desk Reality: Factor Tracking**
>
> Markit publishes the official index factor daily. Risk systems must incorporate this factor to correctly compute:
> - **Notional exposure** — your $100mm trade is only $98.4mm exposure after 2 defaults
> - **Premium payments** — you pay coupon only on outstanding notional
> - **DV01** — index DV01 scales with factor
>
> A common P&L break occurs when systems fail to update the factor after a credit event. The trade shows wrong notional until reconciliation catches it.

### 45.2.6 Clean vs. Full (Dirty) Pricing

Just as with bonds, CDS indices have accrued premium mechanics:

$$\text{Clean MTM} = \text{Full MTM} - \text{Accrued Premium}$$

O'Kane notes that for a **long protection position**, accrued is negative (you owe the accrued premium when transferring the position mid-period).

**Accrued premium formula:**

$$\text{Accrued} = N_{\text{out}} \times C \times \Delta(t_{\text{last}}, t)$$

where $\Delta(t_{\text{last}}, t)$ is the accrual fraction (typically Actual/360) since the last coupon date.

---

## 45.3 Index Lifecycle: From Launch to Maturity

### 45.3.1 Series Launch and Roll Dates

O'Kane describes the roll schedule: "Issue dates typically fall on one of the CDS 'IMM' dates, i.e. 20 March, 20 June, 20 September or 20 December."

Hull confirms: "These portfolios are updated on March 20 and September 20 each year."

**The roll calendar:**

| Issue Date | Maturity Alignment | Next Roll |
|------------|-------------------|-----------|
| March 20 | Matures June 20 (T years out) | September 20 |
| September 20 | Matures December 20 (T years out) | March 20 |

### 45.3.2 What Happens at Roll

O'Kane explains that at roll, "investors who hold the existing on-the-run index will generally try to roll into the new on-the-run index by selling the old index contract and buying the new one."

> **Analogy: The Rolling Train**
>
> Imagine a train (Liquidity) that switches tracks every 6 months.
>
> *   **On-the-Run (e.g., Series 42)**: The train is here. Bid-offer is tight and there are more two-way markets.
> *   **The Roll (Mar 20)**: The train switches to the Series 43 track.
> *   **The Choices**:
>     1.  **Roll**: Jump to Series 43 (Pay transaction costs). Stay liquid.
>     2.  **Stay**: You are now "Off-the-Run." The station is emptier. Bid-offer is wider and exits can be more costly.
> *   **The Force**: Traders *must* roll to maintain the ability to exit. This concentrates volume around Mar 20/Sep 20.

> **Desk Reality: The Roll Period Frenzy**
>
> The two weeks surrounding each roll date see extraordinary activity:
> - **Volume surge:** The roll period accounts for a substantial portion of annual index volume as every holder of the old series must decide whether to roll
> - **Roll cost:** The price difference between selling off-the-run and buying on-the-run. Traders call this "paying the roll"
> - **Positioning:** Dealers position ahead of roll to capture customer flow
> - **Calendar trades:** Some traders explicitly trade the old-new spread as a relative value position
>
> *Note: Exact volume percentages during roll periods vary by market conditions and are not precisely documented in O'Kane.*

**This "almost certainly" generates P&L from two sources:**

1. **Composition changes:** "The new index will not contain any names which have been downgraded below investment grade or whose liquidity has declined." However, O'Kane clarifies that "this does not mean that the new index will be issued at a tighter spread than the current market spread of the old index since liquidity requirements may result in some names being replaced with wider spread names."

2. **Maturity extension:** "The new index has a longer maturity, by six months, than the previous index. As credit curves tend to be upward sloping, this would tend to cause the new index spread to be wider, all other things being the same."

### 45.3.3 Constituent Changes Between Series

Within a series, "the constituents of a specific issue of an index remain fixed for the lifetime of the index, except when they default in which case they are removed without replacement" (O'Kane).

**Across series**, the constituent list can change due to:
- Downgrade out of investment grade
- Credit events
- Material reduction in liquidity
- New qualifying names added as replacements

### 45.3.4 Settlement Timing

O'Kane describes the standard mechanics: "The contract is entered into on trade date $t$ and is cash settled three days later."

| Event | Timing |
|-------|--------|
| Trade date | T |
| Cash settlement | T+3 |
| First coupon | Next scheduled IMM date |
| Subsequent coupons | Quarterly (Actual/360) |

**Coupon setting at inception:** O'Kane notes that "at inception, the index coupon is set close to the fair-value spread of the index. It is not set exactly equal to the fair-value spread as the index coupon is usually chosen to be a multiple of 5 bp."

### 45.3.5 Index Options and Roll Timing

O'Kane discusses the relationship between index options and the roll cycle. Index options (payer and receiver swaptions) are actively traded, with O'Kane noting that "there is currently active trading in options on all of the traded indices, with most liquidity in the cross-over indices of CDX and iTraxx and the CDX high yield index."

However, **option expiries are typically short-dated**: "Because of the six monthly index roll, option expiries beyond six months are rare since the liquidity of the index drops immediately after the next roll and this can significantly increase hedging costs."

This matters operationally because:
- Index option traders must be aware of the roll calendar
- Long-dated hedging strategies may need to roll options as well as the underlying index
- Off-the-run indices have higher hedging costs due to wider bid-ask spreads

---

## 45.4 Default Handling Within Indices

When a constituent defaults, the index contract must handle the credit event systematically. This section covers the mechanics; Chapter 40 provides the detailed auction process.

### 45.4.1 What Happens When a Constituent Defaults

O'Kane describes the mechanics triggered by a default in an $M$-name index:

> "If there is a default, and assuming that the portfolio consists of $M$ credits, what happens is that:
> 1. the buyer pays $1/M$ of the face value of the contract to the seller in return for delivery of a defaulted asset also on $1/M$ of the contract notional. In situations where the outstanding notional of derivative contracts exceeds the supply of deliverable obligations, a new protocol was introduced by ISDA in 2005 which allows a fallback to cash settlement in which case an auction method is used to determine a cash settlement price.
> 2. The buyer receives the fraction of premium which has accrued from the previous coupon date on the defaulted credit.
> 3. The notional of the contract is reduced by $1/M$. As a result, the absolute amount of spread received on the premium leg is reduced."

### 45.4.2 Default Settlement Timeline

The timeline from credit event to settlement follows the CDS auction framework (detailed in Chapter 40):

| Stage | Timing | Action |
|-------|--------|--------|
| Credit Event | Day 0 | Bankruptcy, failure to pay, or (for iTraxx) restructuring |
| Event Determination Date | Within 60 days | ISDA Determinations Committee confirms credit event |
| Auction Announcement | ~5 business days after | Auction date, deliverables announced |
| Auction Date | ~30 days after event | Two-stage auction determines final price |
| Settlement | T+3 after auction | Cash settlement based on auction final price |

### 45.4.3 Cash Settlement Calculation

From the auction final price $FP$ (value per 100 of deliverable), the protection payment is:

$$\boxed{\text{Protection Payment (per name)} = \frac{N}{M} \times \left(1 - \frac{FP}{100}\right)}$$

**Hull's example:** If auction implies deliverable worth 35 per 100, the cash payoff is 65% of the per-name notional.

### 45.4.4 Accrued Premium at Default

The accrued premium on the defaulted credit is paid:

$$\text{Accrued at Default} = \frac{N}{M} \times C \times \Delta(t_{\text{last coupon}}, \tau)$$

where $\tau$ is the default time.

### 45.4.5 Notional Reduction After Default

After settlement, the index outstanding notional reduces:

$$\boxed{N_{\text{out}}^{\text{new}} = N_{\text{out}}^{\text{old}} \times \left(1 - \frac{1}{M}\right)}$$

For a 125-name index, each default reduces outstanding notional by 0.8%.

> **Deep Dive: The "Big Short" Mechanics**
>
> In the movie *The Big Short*, Dr. Burry waits for defaults. How does the payout actually look?
>
> 1.  **The Wait**: He pays premium (bleed). The index notional is full (\$100m).
> 2.  **The Event**: A name defaults.
> 3.  **The Payout**: He gets paid $(1-R) \times \$800k$ immediately (cash settlement).
> 4.  **The Aftermath**: The index notional shrinks to \$99.2m. He stops paying premium on the dead name.
> 5.  **The End Game**: If 10 names default, he collects $\sim\$6m$ in payouts, and the index shrinks. He doesn't get "The Jackpot" all at once; he gets it name-by-name interaction.

**Impact on future coupons:**

$$\text{Coupon}_{\text{new}} = N_{\text{out}}^{\text{new}} \times C \times \alpha$$

where $\alpha$ is the accrual fraction for the period.

### 45.4.6 Regional Differences in Credit Event Definitions

O'Kane notes an important difference: "While the European iTraxx indices include a restructuring credit event, the North American CDX index protection leg is only triggered by a bankruptcy or failure to pay on a reference credit. Restructuring is not included as a credit event."

**Table 45.4: Credit Event Definitions by Region**

| Index Family | Bankruptcy | Failure to Pay | Restructuring | Convention Name |
|--------------|------------|----------------|---------------|-----------------|
| CDX.NA.IG | Yes | Yes | **No** | No-Re (XR) |
| CDX.NA.HY | Yes | Yes | **No** | No-Re (XR) |
| iTraxx Europe | Yes | Yes | **Yes** (Mod-Mod-Re) | MM |
| iTraxx Crossover | Yes | Yes | **Yes** (Mod-Mod-Re) | MM |

**Economic rationale for the difference:**

The No-Re vs. Mod-Re distinction reflects different legal frameworks:

1. **North American practice:** U.S. Chapter 11 bankruptcy provides a well-defined restructuring process. When a company is in distress, it typically files for bankruptcy rather than negotiating an out-of-court restructuring. Including restructuring as a trigger would create ambiguity about what constitutes a credit event.

2. **European practice:** European restructuring norms historically involved more out-of-court debt workouts. Excluding restructuring would leave protection buyers exposed to significant credit deterioration without a payout.

O'Kane quantifies the impact: "The CDX is therefore consistent with the No-Re category of CDS which trades at a spread which is lower than the spread of the standard Mod-Re restructuring clause CDS." He notes that No-Re spreads are "typically about 5% lower than Mod-Re spreads."

**Basis implications:**

This creates structural basis between indices and single-name CDS:
- If single-name contracts trade with Mod-Re but the index is No-Re, there's a mismatch
- A restructuring event triggers single-name CDS but NOT the CDX index
- Hedgers must account for this "restructuring basis" when using indices to hedge single-name exposure

---

## 45.5 Intrinsic Spread and Index Basis: Preview

Chapter 46 provides the detailed treatment of intrinsic spread calculation and index basis trading. Here we preview the core concepts since they are essential for understanding index valuation.

### 45.5.1 The Intrinsic Spread Concept

The **intrinsic spread** of an index is the theoretical spread implied by its constituents. O'Kane defines it (Equation 10.9) as the RPV01-weighted average of constituent spreads:

$$\boxed{S_I^{\text{intr}}(t,T) \approx \frac{\sum_{m=1}^{M} S_m(t,T) \cdot \text{RPV01}_m(t,T)}{\sum_{m=1}^{M} \text{RPV01}_m(t,T)}}$$

where:
- $S_m$ = single-name spread for constituent $m$
- $\text{RPV01}_m$ = risky PV01 for constituent $m$

**Why RPV01-weighted, not equal-weighted?**

A simple average of constituent spreads would be incorrect because each constituent has different credit risk, hence different RPV01. The intrinsic spread must be weighted by the amount of premium each name contributes to the index.

### 45.5.2 Index Basis

The **index basis** is the difference between the quoted index spread and the intrinsic spread:

$$\boxed{\text{Index Basis} = S_I^{\text{quoted}} - S_I^{\text{intrinsic}}}$$

O'Kane discusses why the basis can be positive or negative:

1. **No-Re vs. Mod-Re mismatch:** "Since the iTraxx indices are Mod-Mod-Re and the CDX indices are No-Re, there can be a difference between the index spread and the intrinsic spread."

2. **Liquidity premium:** Index liquidity may command a premium (tighter spread) versus single-name average.

3. **Market-leading behavior:** In stressed markets, the index may move before single-name spreads adjust.

**Chapter 46** covers basis calculation in full detail, including worked examples and trading strategies.

---

## 45.6 Worked Examples

### Example 45.A: Parse a Complete Index Identifier

**Task:** Decode "CDX.NA.HY.7 5Y" and identify all relevant parameters.

**Solution:**

| Component | Parsed Value | Meaning |
|-----------|--------------|---------|
| CDX | Index family | North American CDS index family |
| NA | Region | North America |
| HY | Credit quality | High Yield |
| 7 | Series | 7th vintage (series numbering; date depends on index family) |
| 5Y | Tenor label | 5-year maturity contract |

**Additional facts from Table 45.1:**
- Number of credits: 100 (standard for CDX.NA.HY)
- Quoting convention: Bond price (not spread)
- Fixed coupon: Fixed at series inception (series-specific; do not assume)
- Roll dates: March 20 and September 20

---

### Example 45.B: Convert Between Spread and Price Quoting

**Given:** CDX.NA.IG Series 7
- Coupon $C = 40$ bp $= 0.0040$
- Market spread $S_I = 34$ bp $= 0.0034$
- $\text{RPV01}_I = 4.447$ years

**Task:** Calculate the bond-equivalent price.

**Step 1:** Calculate upfront as percent of notional:

$$U_{\%} = (S_I - C) \times \text{RPV01}_I$$
$$U_{\%} = (0.0034 - 0.0040) \times 4.447$$
$$U_{\%} = (-0.0006) \times 4.447 = -0.002668 = -0.2668\%$$

**Step 2:** Convert to bond price:

$$P_{\text{bond}} = 100\% - (-0.2668\%) = 100.27\%$$

**Interpretation:** The negative upfront means the protection buyer receives this amount (or equivalently, the index trades above par at 100.27%). This matches Hull's Example 25.1 and O'Kane's Table 10.2.

**Unit check:** bp × years = dimensionless (percent of notional) ✓

---

### Example 45.C: Calculate Index Notional and Coupon After Defaults

**Given:** CDX.NA.IG with:
- Initial notional $N = \$100,000,000$
- Number of constituents $M = 125$
- Coupon $C = 100$ bp $= 0.0100$
- Quarterly accrual fraction $\alpha = 0.25$
- **Two defaults have occurred**

**Task:** Calculate outstanding notional, index factor, and quarterly coupon.

**Step 1:** Index factor:

$$f = \frac{M - D}{M} = \frac{125 - 2}{125} = \frac{123}{125} = 0.984$$

**Step 2:** Outstanding notional:

$$N_{\text{out}} = N \times f = 100,000,000 \times 0.984 = \$98,400,000$$

**Step 3:** Quarterly coupon payment:

**Before defaults:**
$$\text{Coupon} = 100,000,000 \times 0.0100 \times 0.25 = \$250,000$$

**After 2 defaults:**
$$\text{Coupon} = 98,400,000 \times 0.0100 \times 0.25 = \$246,000$$

**Reduction:** $\$250,000 - \$246,000 = \$4,000$ per quarter

---

### Example 45.D: Default Settlement Calculation

**Given:**
- Index notional $N = \$50,000,000$
- Number of names $M = 125$
- One constituent defaults
- Auction final price $FP = 35$ (per 100 face value)

**Task:** Calculate the protection payment.

**Step 1:** Per-name notional:

$$N_{\text{name}} = \frac{N}{M} = \frac{50,000,000}{125} = \$400,000$$

**Step 2:** Loss given default (LGD) fraction:

$$\text{LGD fraction} = 1 - \frac{FP}{100} = 1 - 0.35 = 0.65 = 65\%$$

**Step 3:** Protection payment:

$$\text{Payment} = N_{\text{name}} \times \text{LGD fraction} = 400,000 \times 0.65 = \$260,000$$

**Sanity checks:**
- If $FP = 0$ (total loss): Payment $= \$400,000$ ✓ (bounded by per-name notional)
- If $FP = 100$ (full recovery): Payment $= \$0$ ✓

---

### Example 45.E: Roll Trade Mechanics

**Scenario:** You hold long protection on off-the-run CDX.NA.IG Series 6 and want to roll to on-the-run Series 7.

**Given:**
- Notional $N = \$25,000,000$
- Series 6 (off-the-run) upfront to unwind: you receive 0.80% to close
- Series 7 (on-the-run) upfront to enter: you pay 0.50%

**Task:** Calculate net cash from roll.

**Step 1:** Cash from unwinding Series 6:

To close long protection, you enter offsetting short protection. You receive:
$$\text{Receive} = 25,000,000 \times 0.0080 = \$200,000$$

**Step 2:** Cash to enter Series 7:

To enter new long protection, you pay:
$$\text{Pay} = 25,000,000 \times 0.0050 = \$125,000$$

**Step 3:** Net roll cash:

$$\text{Net} = \$200,000 - \$125,000 = +\$75,000 \text{ (received)}$$

**Note:** This ignores bid/ask spreads and assumes both trades settle on the same accrual grid. In practice, off-the-run bid/ask is wider (O'Kane notes liquidity drops after roll), so actual roll may cost more.

---

### Example 45.F: Impact of Quoting Convention Error

**Scenario:** A trader mistakenly books a CDX.NA.HY trade using spread convention instead of price convention.

**Given:**
- Notional $N = \$100,000,000$
- Correct quote: Bond price = 102.18%
- Incorrectly assumed: Spread quote treated as coupon + upfront

**What's the error?**

**Correct (price convention):**
$$U_{\%} = 100\% - 102.18\% = -2.18\%$$
$$U_{\$} = 100,000,000 \times 0.0218 = \$2,180,000 \text{ received}$$

If you mistakenly treat the bond price as an upfront quote, you might book +2.18% paid instead of -2.18% received — a $4.36mm cashflow difference on $100mm notional.

**If spread were misinterpreted:**
Suppose the trader thought "102.18" was a spread in bp and applied spread mechanics with a different result — the error could be millions of dollars.

**Lesson:** Always verify the quoting convention for the specific index family before booking. CDX.NA.HY uses bond price; CDX.NA.IG uses spread.

---

### Example 45.G: Index Factor Impact on DV01

**Given:**
- CDX.NA.IG position: $100mm notional, 5Y maturity
- Index DV01 (full factor): $47,000 per bp
- Current factor: $f = 0.976$ (3 defaults have occurred)

**Task:** Calculate actual DV01 exposure.

**Solution:**

$$\text{Actual DV01} = \text{Full DV01} \times f = 47,000 \times 0.976 = \$45,872$$

**Risk implication:** If your risk system shows $47,000 DV01 but the factor is 0.976, your actual exposure is $1,128 less per bp. Over a 10 bp move, that's an $11,280 P&L discrepancy.

---

## 45.7 Practical Notes

### 45.7.1 Booking Checklist for Index Trades

- [ ] **Index identifier:** Family, region, quality, series number
- [ ] **Tenor:** 3Y/5Y/7Y/10Y (note actual maturity is ±3 months from label)
- [ ] **On-the-run vs. off-the-run:** Affects liquidity and roll timing
- [ ] **Quoting convention:** Spread (IG) vs. price (HY/EM)
- [ ] **Fixed coupon:** Series-specific; confirm from confirmation/reference data
- [ ] **Upfront:** Amount and direction (pay/receive)
- [ ] **Settlement date:** Typically T+3 from trade date
- [ ] **Current index factor:** Number of names removed and current factor
- [ ] **Accrued premium:** If trading between coupon dates

### 45.7.2 Common Pitfalls

1. **Quoting convention confusion:** IG indices quote spread; HY/EM quote price. Mixing them causes large booking errors.

2. **Series/tenor confusion:** "Series 7 5Y" means the 7th vintage of the 5-year contract, not a 7-year contract in Series 5.

3. **Forgetting the ±3 month rule:** A "5Y" index at issuance has ~63 months to maturity, not 60.

4. **Roll date assumptions:** Verify actual roll schedule — market conventions can shift.

5. **Ignoring the index factor:** The index may have fewer than 125 names if defaults have occurred. Notional, DV01, and coupon all scale with the factor.

6. **Regional credit event differences:** CDX excludes restructuring (No-Re); iTraxx includes it (Mod-Re). This affects basis calculations and hedging.

7. **Hedging off-the-run positions:** O'Kane warns that "if we hedge an off-the-run index swap position with an on-the-run index swap position, there may be a mismatch in terms of some credits being in one index but not in the other."

### 45.7.3 Verification Tests

1. **Price-spread consistency:** $P_{\text{bond}} = 100 - 100 \\times \\text{RPV01}_I \\times (S_I - C)$ should match market quote

2. **Factor bounds:** $f = (M - D)/M$ must be in range $(0, 1]$

3. **Notional consistency:** $N_{\text{out}} = N \times f$

4. **Payment bounds:** Protection payment per default $\leq N/M$

5. **Coupon scaling:** Coupon payments scale linearly with outstanding notional

---

## Summary

CDS indices provide efficient exposure to diversified credit portfolios through standardized contracts. The key operational considerations are:

1. **Naming conventions** encode geography, credit quality, series (vintage), and tenor — each component affects liquidity and pricing

2. **Quoting conventions** differ by index family: investment grade often quotes by spread; high yield/EM often quote by bond price — confusion causes booking errors

3. **The ±3 month rule** means a "5Y" index at issuance is actually ~5.25 years to maturity

4. **Series roll every six months** (March 20 and September 20), with liquidity concentrating in on-the-run

5. **The index factor** $f = (M-D)/M$ tracks notional reduction after defaults — critical for accurate DV01 and P&L

6. **Defaults reduce outstanding notional by 1/M**, lowering future coupon payments proportionally

7. **Cash settlement via auction** determines recovery; protection payment is $(1 - FP/100)$ times per-name notional

8. **Regional differences** in credit event definitions (CDX = No-Re, iTraxx = Mod-Re) create basis versus single-name CDS — No-Re spreads are typically ~5% lower

9. **Intrinsic spread** is the RPV01-weighted average of constituent spreads; index basis reflects the difference from quoted spread

10. **Exceptional liquidity** of the major indices — O'Kane cites bid-offer spreads as tight as 0.25 bp for IG indices

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **On-the-run** | Most recently issued series | Concentrates liquidity; benchmark for hedging |
| **Off-the-run** | Previous series (still tradable) | Lower liquidity; may have different constituents |
| **Series** | A specific vintage with fixed coupon and constituent list | Identifies which version you're trading |
| **Index factor ($f$)** | $(M-D)/M$ — fraction of names remaining | Scales notional, DV01, and coupons |
| **Bond price convention** | Quote as price (100 = par); upfront = price - 100 | Used for HY/EM; avoids PV01 disputes |
| **Spread convention** | Quote as spread; upfront = (spread - coupon) × RPV01 | Used for IG indices |
| **Roll** | Transitioning from old to new series | Causes P&L from composition and maturity changes |
| **Notional reduction** | Outstanding notional drops by 1/M per default | Affects future coupon amounts |
| **Flat curve convention** | Index priced using flat term structure at quoted spread | Standard market convention for index valuation |
| **No-Re (XR)** | No restructuring credit event (CDX convention) | Spread ~5% lower than Mod-Re |
| **Mod-Mod-Re (MM)** | Restructuring included (iTraxx convention) | Broader credit event coverage |
| **Intrinsic spread** | RPV01-weighted average of constituent spreads | Basis = quoted - intrinsic |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What does "CDX.NA.IG.42 5Y" mean? | CDX index, North America, Investment Grade, Series 42, 5-year maturity |
| 2 | How many constituents in CDX.NA.IG? | 125 |
| 3 | How many constituents in CDX.NA.HY? | 100 |
| 4 | What is "on-the-run"? | The most recently issued series (concentrates liquidity) |
| 5 | How often do CDS indices roll? | Every six months (March 20 and September 20) |
| 6 | Does a "5Y" index have exactly 5 years to maturity at issue? | No — typically T+3 months at issuance, T-3 months by next roll |
| 7 | How does CDX.NA.IG quote in the market? | By spread (not price) |
| 8 | How does CDX.NA.HY quote in the market? | By bond price (not spread) |
| 9 | Why do HY indices use bond price quoting? | To avoid disputes about index PV01 for spread-to-upfront conversion |
| 10 | Convert bond price 102.18% to upfront | Upfront = 100% - 102.18% = -2.18% (you receive) |
| 11 | Convert bond price 99.50% to upfront | Upfront = 100% - 99.50% = +0.50% (you pay) |
| 12 | Formula: Upfront from spread | $U_I = (S_I - C) \times \text{RPV01}_I$ |
| 13 | If spread < coupon, what happens to upfront? | Upfront is negative (protection buyer receives payment) |
| 14 | What happens to index notional when a constituent defaults? | Reduced by 1/M (e.g., 0.8% for 125-name index) |
| 15 | Cash settlement payment per default formula | Payment = $(N/M) \times (1 - FP/100)$ |
| 16 | If auction final price = 35, what is LGD percentage? | 65% (protection payment = 65% of per-name notional) |
| 17 | What credit events trigger CDX protection? | Bankruptcy and failure to pay (NOT restructuring) |
| 18 | What credit events trigger iTraxx protection? | Bankruptcy, failure to pay, AND restructuring |
| 19 | What causes P&L at roll (two reasons)? | 1) Composition changes; 2) Maturity extension by 6 months |
| 20 | What day count is used for index coupons? | Actual/360, paid quarterly |
| 21 | How is coupon set at index inception? | Close to fair-value spread, rounded to multiple of 5 bp |
| 22 | Why does liquidity matter at roll? | Liquidity drops immediately after roll for off-the-run |
| 23 | What is "accrued premium at default"? | Premium owed from last coupon date to default date |
| 24 | Why do off-the-run series have hedge mismatch risk? | Constituents may differ from on-the-run |
| 25 | Settlement timing for index trades? | T+3 (cash settled three days after trade date) |
| 26 | What is the typical bid-offer spread for IG indices? | ~0.25 bp (exceptionally tight due to high liquidity) |
| 27 | Who administers the CDX and iTraxx indices? | Markit Group Limited (now S&P Global) |
| 28 | Why are index option expiries beyond 6 months rare? | Liquidity drops after roll, increasing hedging costs |
| 29 | Formula for index factor | $f = (M - D) / M$ where M = original names, D = defaults |
| 30 | What is "No-Re" (XR)? | No restructuring credit event — CDX convention |
| 31 | What is "Mod-Mod-Re" (MM)? | Modified-modified restructuring included — iTraxx convention |
| 32 | By how much are No-Re spreads typically lower than Mod-Re? | ~5% lower (per O'Kane) |
| 33 | What is intrinsic spread? | RPV01-weighted average of constituent single-name spreads |
| 34 | What is index basis? | Quoted index spread minus intrinsic spread |
| 35 | What is the fixed coupon for CDX.NA.IG? | Series-specific (fixed at inception); confirm on confirmation/reference data |

---

## Mini Problem Set

**Questions 1-6: Solution sketches provided**

**1.** Parse the identifier "iTraxx Europe HiVol Series 6 7Y" and list all components.

> **Sketch:** iTraxx (family), Europe (region), HiVol (high volatility subset), Series 6 (vintage), 7Y (maturity label). Number of names: 30. Credit event convention: Mod-Mod-Re.

**2.** CDX.NA.IG Series 7: Coupon = 40 bp, spread = 48 bp, RPV01 = 4.0. Calculate upfront % and whether you pay or receive.

> **Sketch:** $U = (0.0048 - 0.0040) \times 4.0 = 0.0032 = 0.32\%$. Spread > coupon → you pay 0.32% to enter long protection.

**3.** CDX.NA.HY bond price = 103.50%. Calculate dollar upfront on $50mm notional.

> **Sketch:** $U_{\%} = 100 - 103.50 = -3.50\%$. $U_{\$} = 50,000,000 \times 0.035 = \$1,750,000$ received.

**4.** An index has notional $100mm, 125 names, coupon 80 bp, quarterly α = 0.25. After 3 defaults, what is the index factor and quarterly coupon payment?

> **Sketch:** $f = 122/125 = 0.976$. $N_{\text{out}} = 100mm \times 0.976 = \$97.6mm$. Coupon = $97,600,000 \times 0.008 \times 0.25 = \$195,200$.

**5.** Auction final price = 22. Calculate protection payment per name if per-name notional is $800,000.

> **Sketch:** LGD = 1 - 0.22 = 0.78. Payment = $800,000 \times 0.78 = \$624,000$.

**6.** At issuance on March 20 (an IMM date), a "5Y" CDX.NA.IG matures on what date? How many months to maturity at issuance?

> **Sketch:** Matures June 20 five years later. Months = 63 (5 years + 3 months).

**Questions 7-12: No sketches**

**7.** You roll $30mm from off-the-run (receive 0.45% to close) to on-the-run (pay 0.20% to enter). Calculate net roll cash.

**8.** Explain why CDX.NA.IG and iTraxx Europe can have basis versus their single-name constituents due to restructuring clause differences.

**9.** After 5 defaults in a 100-name index, what is the index factor and what fraction of original notional remains?

**10.** An index has current outstanding notional of $94.4mm after defaults. If original notional was $100mm and M = 125, how many defaults have occurred?

**11.** Why are index option expiries beyond 6 months rare?

**12.** A CDX.NA.IG position shows DV01 of $50,000 at full factor. If the current factor is 0.968, what is the actual DV01 exposure?

---

## References

- Dominic O'Kane, *Modelling Single-name and Multi-name Credit Derivatives* (CDS portfolio indices: quoting, roll mechanics, default handling)
- John C. Hull, *Options, Futures, and Other Derivatives* (credit indices and bond-price/intuitive pricing examples)

## Cross-References

- **Chapter 40** — CDS Auction Process: Detailed two-stage auction mechanics for default settlement
- **Chapter 41** — CDS Index Pricing: Economics and cashflow valuation (premium/protection legs)
- **Chapter 46** — Intrinsic Spread and Index Basis: Full treatment of basis calculation and trading strategies
- **Chapter 47** — Index Hedging: Managing index versus single-name hedge mismatches
- **Chapters 48-51** — Tranche Products: CDO tranches built on index portfolios
