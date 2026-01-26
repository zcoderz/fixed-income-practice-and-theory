# Chapter 45: CDS Indices — Structure, Quoting, and Lifecycle

---

## Introduction

"CDX.NA.IG.42 5Y" — what does each part of this cryptic string mean, and why does misunderstanding it matter?

When a trader says they're "buying CDX," they're entering a single contract that provides credit exposure to 125 investment-grade North American companies simultaneously. This is the power of CDS indices: one trade to express a macro credit view, one instrument to hedge a diverse credit book, and — crucially — far more liquidity than trading 125 single-name CDS contracts individually. O'Kane emphasizes that "CDS portfolio indices have transformed the credit derivatives markets by providing a highly liquid product which investors can use to assume or hedge the risk of a broad portfolio of credit exposures." But this convenience comes with complexity. The naming convention encodes geography, credit quality, vintage, and tenor. The quoting convention shifts between spread and price depending on the index family. And the lifecycle — from new series launch through constituent defaults to roll dates — creates operational landmines for the unwary.

**Why this matters:** Consider a trader who books a CDX.NA.HY trade using the wrong quoting convention. High-yield indices quote as "bond price" (not spread), so confusion between the two conventions can cause upfront payment errors of several percent of notional — on a $100 million trade, that's millions of dollars. Similarly, failing to track which series is "on-the-run" versus "off-the-run" affects liquidity, hedge effectiveness, and roll P&L. These aren't theoretical concerns; they're daily operational realities on credit trading desks.

**Chapter roadmap:** This chapter covers the **market structure and operational mechanics** of CDS indices:

- **Section 45.1** decodes index naming conventions — the systematic structure behind identifiers like "iTraxx Europe Series 6"
- **Section 45.2** explains quoting conventions — when indices quote as spread versus price, and how to convert between them
- **Section 45.3** covers the index lifecycle — from series launch through rolls to maturity
- **Section 45.4** addresses default handling within indices — the timeline from credit event to auction to settlement
- **Section 45.5** presents worked examples for common operational calculations

**Relationship to other chapters:** Chapter 41 covers the *economics* of CDS indices — coupon mechanics, premium leg valuation, and the upfront-spread relationship. This chapter focuses on *market structure and operations*. Chapter 46 addresses the intrinsic spread calculation and index basis. Chapter 40 provides the detailed CDS auction mechanics that we reference for default settlement.

---

## Notation for This Chapter

| Symbol | Definition |
|--------|------------|
| $N$ | Index trade notional (USD) |
| $M$ | Number of index constituents (e.g., 125 for CDX.NA.IG) |
| $C$ or $c$ | Annualized running coupon rate in decimal (e.g., 100 bp $= 0.01$) |
| $S_I(t,T)$ | Market-quoted flat index spread at time $t$ for maturity $T$ |
| $U_I$ | Upfront payment (percent of notional; can be negative) |
| $\text{RPV01}_I$ | Risky PV01 of the index (PV of 1 bp running premium) |
| $P_{\text{bond}}$ | Bond-price quote (as percent of notional, e.g., 102.18%) |
| $FP$ | Auction final price (value per 100 of deliverable) used for cash settlement |

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

> **Tradable vs. benchmark indices:** O'Kane distinguishes CDS portfolio indices from benchmark indices: "While the CDS indices are designed with the primary aim of tradability, benchmark indices are more concerned with providing an accurate reflection of the performance of the 'market'."

### 45.1.5 Series and Version: The Vintage System

O'Kane explains that "most CDS indices are issued or rolled semi-annually with a coupon which is fixed for the lifetime of the issue." Each issuance creates a new **series** with a fixed constituent list (except for default removals).

**Series numbering:**
- CDX.NA.IG Series 35 was defined in September 2020 (per Hull)
- iTraxx Europe Series 34 was defined in September 2020
- The series number indicates how many times the index has been "refreshed" since inception

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

**Table 45.3: Quoting Convention by Index Family**

| Index Family | Typical Quoting Convention | Reason |
|--------------|---------------------------|--------|
| CDX.NA.IG | Spread | Low PV01 dispute risk |
| iTraxx Europe | Spread | Low PV01 dispute risk |
| CDX.NA.HY | Bond Price | Avoids PV01 disputes |
| CDX.EM | Bond Price | Avoids PV01 disputes |
| iTraxx Crossover | Bond Price | Higher credit risk |

### 45.2.2 Spread Quoting (Investment Grade)

When an index quotes by spread, the **upfront payment** is derived from the difference between the quoted spread and the contractual coupon, scaled by the risky PV01:

$$\boxed{U_I = (S_I - C) \times \text{RPV01}_I}$$

**Interpretation:**
- If $S_I > C$: Spread has widened since issuance → investor entering receives upfront (index "trades below par")
- If $S_I < C$: Spread has tightened → investor entering pays upfront (index "trades above par")

**The flat curve convention:** O'Kane explains that "market convention dictates that the index curve used to value an index has a flat term structure. So if we wish to value the 7Y CDX NA Investment Grade index, we would simply use a flat CDS curve at a level of 44 bp." This is important: the $\text{RPV01}_I$ in the formula above is computed using a flat spread assumption, not a term structure.

**Example from O'Kane (Table 10.2, November 2006):**

| Index | Series | Coupon (bp) | Index Spread (bp) | Implied Bond Price |
|-------|--------|-------------|-------------------|-------------------|
| CDX.NA.IG | 7 | 40 | 34 | 100.27% |
| iTraxx Europe | 6 | 30 | 24 | 100.28% |

The CDX.NA.IG spread (34 bp) is below the coupon (40 bp), meaning credit quality has improved since issuance. An investor entering must pay upfront (reflected in the bond price above 100).

### 45.2.3 Bond Price Quoting (High Yield / EM)

O'Kane explains the rationale: "this convention has the advantage that it avoids any disagreement about the value of the index PV01 which is required to convert a spread-based quotation into an upfront value."

**The conversion is simple:**

$$\boxed{U_{\%} = P_{\text{bond}} - 100\%}$$

**Examples:**
- Bond price = 102.18% → Pay 2.18% upfront
- Bond price = 99.50% → Receive 0.50% upfront

**Dollar upfront from bond price:**

$$\boxed{U_{\$} = N \times (P_{\text{bond}}/100 - 1)}$$

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

### 45.2.5 Clean vs. Full (Dirty) Pricing

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
> *   **On-the-Run (Series 42)**: The train is here. Spreads are tight (0.5bp). Everyone is trading.
> *   **The Roll (Mar 20)**: The train switches to the Series 43 track.
> *   **The Choices**:
>     1.  **Roll**: Jump to Series 43 (Pay transaction costs). Stay liquid.
>     2.  **Stay**: You are now "Off-the-Run." The station is empty. Spreads widen (5bp). You are stuck with an illiquid position.
> *   **The Force**: Traders *must* roll to maintain the ability to exit. This creates massive volume on Mar 20/Sep 20.

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

This means CDX trades as "No-Re" (no restructuring), while iTraxx typically includes restructuring (Mod-Re or Old-Re). O'Kane adds: "The CDX is therefore consistent with the No-Re category of CDS which trades at a spread which is lower than the spread of the standard Mod-Re restructuring clause CDS." This creates basis between indices and their single-name constituents when single-name contracts include restructuring.

---

## 45.5 Worked Examples

### Example 45.A: Parse a Complete Index Identifier

**Task:** Decode "CDX.NA.HY.7 5Y" and identify all relevant parameters.

**Solution:**

| Component | Parsed Value | Meaning |
|-----------|--------------|---------|
| CDX | Index family | North American CDS index family |
| NA | Region | North America |
| HY | Credit quality | High Yield |
| 7 | Series | 7th vintage (issued ~2007) |
| 5Y | Tenor label | 5-year maturity contract |

**Additional facts from Table 45.1:**
- Number of credits: 100 (standard for CDX.NA.HY)
- Quoting convention: Bond price (not spread)
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

**Interpretation:** The negative upfront means the investor entering receives this amount (or equivalently, the index trades above par at 100.27%). This matches Hull's Example 25.1 and O'Kane's Table 10.2.

**Unit check:** bp × years = dimensionless (percent of notional) ✓

---

### Example 45.C: Calculate Index Notional and Coupon After Defaults

**Given:** CDX.NA.IG with:
- Initial notional $N = \$100,000,000$
- Number of constituents $M = 125$
- Coupon $C = 100$ bp $= 0.0100$
- Quarterly accrual fraction $\alpha = 0.25$
- **Two defaults have occurred**

**Task:** Calculate outstanding notional and quarterly coupon.

**Step 1:** Notional reduction per default:

$$\Delta N = \frac{N}{M} = \frac{100,000,000}{125} = \$800,000$$

**Step 2:** Outstanding notional after 2 defaults:

$$N_{\text{out}} = N - 2 \times \Delta N = 100,000,000 - 2 \times 800,000 = \$98,400,000$$

Alternatively:
$$N_{\text{out}} = N \times \left(1 - \frac{2}{M}\right) = 100,000,000 \times \frac{123}{125} = \$98,400,000$$

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
$$U_{\%} = 102.18\% - 100\% = +2.18\%$$
$$U_{\$} = 100,000,000 \times 0.0218 = \$2,180,000 \text{ paid}$$

**If spread were misinterpreted:**
Suppose the trader thought "102.18" was a spread in bp and applied spread mechanics with a different result — the error could be millions of dollars.

**Lesson:** Always verify the quoting convention for the specific index family before booking. CDX.NA.HY uses bond price; CDX.NA.IG uses spread.

---

## 45.6 Practical Notes

### 45.6.1 Booking Checklist for Index Trades

- [ ] **Index identifier:** Family, region, quality, series number
- [ ] **Tenor:** 3Y/5Y/7Y/10Y (note actual maturity is ±3 months from label)
- [ ] **On-the-run vs. off-the-run:** Affects liquidity and roll timing
- [ ] **Quoting convention:** Spread (IG) vs. price (HY/EM)
- [ ] **Coupon:** Fixed contractual spread for the series (typically multiples of 5 bp)
- [ ] **Upfront:** Amount and direction (pay/receive)
- [ ] **Settlement date:** Typically T+3 from trade date
- [ ] **Current defaults:** Number of names removed and current outstanding notional
- [ ] **Accrued premium:** If trading between coupon dates

### 45.6.2 Common Pitfalls

1. **Quoting convention confusion:** IG indices quote spread; HY/EM quote price. Mixing them causes large booking errors.

2. **Series/tenor confusion:** "Series 7 5Y" means the 7th vintage of the 5-year contract, not a 7-year contract in Series 5.

3. **Forgetting the ±3 month rule:** A "5Y" index at issuance has ~63 months to maturity, not 60.

4. **Roll date assumptions:** Verify actual roll schedule — market conventions can shift.

5. **Constituent count after defaults:** The index may have fewer than 125 names if defaults have occurred.

6. **Regional credit event differences:** CDX excludes restructuring (No-Re); iTraxx includes it (Mod-Re). This affects basis calculations.

7. **Hedging off-the-run positions:** O'Kane warns that "if we hedge an off-the-run index swap position with an on-the-run index swap position, there may be a mismatch in terms of some credits being in one index but not in the other."

### 45.6.3 Verification Tests

1. **Price-spread consistency:** $P_{\text{bond}} = 100 + (S_I - C) \times \text{RPV01}_I$ should match market quote

2. **Notional bounds:** After $d$ defaults: $N_{\text{out}} = N \times (1 - d/M)$

3. **Payment bounds:** Protection payment per default $\leq N/M$

4. **Coupon scaling:** Coupon payments scale linearly with outstanding notional

---

## Summary

CDS indices provide efficient exposure to diversified credit portfolios through standardized contracts. The key operational considerations are:

1. **Naming conventions** encode geography, credit quality, series (vintage), and tenor — each component affects liquidity and pricing

2. **Quoting conventions** differ by index family: investment grade quotes by spread, high yield/EM quote by bond price — confusion causes booking errors

3. **The ±3 month rule** means a "5Y" index at issuance is actually ~5.25 years to maturity

4. **Series roll every six months** (March 20 and September 20), with liquidity concentrating in on-the-run

5. **Defaults reduce outstanding notional by 1/M**, lowering future coupon payments proportionally

6. **Cash settlement via auction** determines recovery; protection payment is $(1 - FP/100)$ times per-name notional

7. **Regional differences** in credit event definitions (CDX = No-Re, iTraxx = Mod-Re) create basis versus single-name CDS

8. **Exceptional liquidity** of the major indices — O'Kane cites bid-offer spreads as tight as 0.25 bp for IG indices

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **On-the-run** | Most recently issued series | Concentrates liquidity; benchmark for hedging |
| **Off-the-run** | Previous series (still tradable) | Lower liquidity; may have different constituents |
| **Series** | A specific vintage with fixed coupon and constituent list | Identifies which version you're trading |
| **Bond price convention** | Quote as price (100 = par); upfront = price - 100 | Used for HY/EM; avoids PV01 disputes |
| **Spread convention** | Quote as spread; upfront = (spread - coupon) × RPV01 | Used for IG indices |
| **Roll** | Transitioning from old to new series | Causes P&L from composition and maturity changes |
| **Notional reduction** | Outstanding notional drops by 1/M per default | Affects future coupon amounts |
| **Flat curve convention** | Index priced using flat term structure at quoted spread | Standard market convention for index valuation |

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
| 10 | Convert bond price 102.18% to upfront | Upfront = 102.18% - 100% = +2.18% (you pay) |
| 11 | Convert bond price 99.50% to upfront | Upfront = 99.50% - 100% = -0.50% (you receive) |
| 12 | Formula: Upfront from spread | $U_I = (S_I - C) \times \text{RPV01}_I$ |
| 13 | If spread < coupon, what happens to upfront? | Upfront is negative (investor receives payment) |
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
| 27 | Who administers the CDX and iTraxx indices? | Markit Group Limited |
| 28 | Why are index option expiries beyond 6 months rare? | Liquidity drops after roll, increasing hedging costs |

---

## Mini Problem Set

**Questions 1-6: Solution sketches provided**

**1.** Parse the identifier "iTraxx Europe HiVol Series 6 7Y" and list all components.

> **Sketch:** iTraxx (family), Europe (region), HiVol (high volatility subset), Series 6 (vintage), 7Y (maturity label). Number of names: 30.

**2.** CDX.NA.IG Series 7: Coupon = 40 bp, spread = 48 bp, RPV01 = 4.0. Calculate upfront % and whether you pay or receive.

> **Sketch:** $U = (0.0048 - 0.0040) \times 4.0 = 0.0032 = 0.32\%$. Spread > coupon → you receive 0.32% to enter long protection.

**3.** CDX.NA.HY bond price = 103.50%. Calculate dollar upfront on $50mm notional.

> **Sketch:** $U_{\%} = 103.50 - 100 = 3.50\%$. $U_{\$} = 50,000,000 \times 0.035 = \$1,750,000$ paid.

**4.** An index has notional $100mm, 125 names, coupon 80 bp, quarterly α = 0.25. After 3 defaults, what is the quarterly coupon payment?

> **Sketch:** $N_{\text{out}} = 100mm \times (122/125)$. Coupon = $N_{\text{out}} \times 0.008 \times 0.25$.

**5.** Auction final price = 22. Calculate protection payment per name if per-name notional is $800,000.

> **Sketch:** LGD = 1 - 0.22 = 0.78. Payment = $800,000 \times 0.78 = \$624,000$.

**6.** At issuance on March 20, 2024, a "5Y" CDX.NA.IG matures on what date? How many months to maturity?

> **Sketch:** Matures June 20, 2029. Months = 63 (5 years + 3 months).

**Questions 7-12: No sketches**

**7.** You roll $30mm from off-the-run (receive 0.45% to close) to on-the-run (pay 0.20% to enter). Calculate net roll cash.

**8.** Explain why CDX.NA.IG and iTraxx Europe can have basis versus their single-name constituents.

**9.** After 5 defaults in a 100-name index, what fraction of original notional remains?

**10.** An index has current outstanding notional of $94.4mm after defaults. If original notional was $100mm and M = 125, how many defaults have occurred?

**11.** Why are index option expiries beyond 6 months rare?

**12.** List three types of constituent changes that can occur at roll.

---

## Source Map

### (A) Verified Facts (Source-Backed)

| Fact | Source |
|------|--------|
| Index families (CDX.NA.IG, iTraxx Europe, etc.) and constituent counts | O'Kane Ch. 10, Table 10.1 |
| "CDS portfolio indices have transformed the credit derivatives markets" | O'Kane Ch. 10, Section 10.1 |
| Roll dates (March 20, September 20) and IMM alignment | O'Kane Ch. 10; Hull Ch. 25 |
| "T-year" indices have T±3 months actual maturity | O'Kane Ch. 10 |
| IG indices quote by spread; HY/EM quote by bond price | O'Kane Ch. 10 |
| Bond price convention avoids PV01 disputes | O'Kane Ch. 10 |
| Upfront = (Spread - Coupon) × RPV01 | O'Kane Eq. 10.7 |
| Default reduces notional by 1/M | O'Kane Ch. 10 |
| Accrued premium paid at default | O'Kane Ch. 10 |
| CDX = No-Re; iTraxx = Mod-Re (restructuring clause difference) | O'Kane Ch. 10 |
| ISDA 2005 auction protocol for cash settlement | O'Kane Ch. 10 |
| Liquidity drops after roll; expiries beyond 6 months rare for options | O'Kane Ch. 10-11 |
| Example price calculation (CDX.NA.IG = 100.27%) | O'Kane Table 10.2; Hull Example 25.1 |
| Bid-offer spread of 0.25 bp for IG indices, ~4 bp for HY | O'Kane Ch. 10 |
| Markit as index administrator | O'Kane Ch. 10 |
| Index coupon set as multiple of 5 bp | O'Kane Ch. 10 |
| Flat curve convention for index pricing | O'Kane Ch. 10 |
| Five advantages of index products (diversification, shorting, capital efficiency, derivatives underlying, sector trading) | O'Kane Ch. 10 |
| Off-the-run hedge mismatch due to constituent differences | O'Kane Ch. 10 |

### (B) Reasoned Inference (Derived from A)

| Inference | Derivation |
|-----------|------------|
| Coupon payment formula after defaults | Linear scaling of coupon × outstanding notional from mechanics |
| Roll P&L from composition + maturity | Direct from O'Kane's two listed drivers |
| Booking error magnitude from quoting confusion | Arithmetic consequence of formula differences |

### (C) Flagged Uncertainties

- **Exact current series numbers:** Series numbers increment with each roll; specific numbers depend on date
- **Exact constituent lists:** Require index rulebook (Markit documentation) for current series
- **Precise settlement conventions:** May vary; verify against trade confirmation and ISDA definitions
- **Desk-specific PV01/MTM conventions:** Require valuation policy documentation
- **Current bid-offer spreads:** O'Kane's figures are from ~2006-2007; current spreads may differ

---

## Cross-References

- **Chapter 40** — CDS Auction Process: Detailed two-stage auction mechanics for default settlement
- **Chapter 41** — CDS Index Pricing: Economics and cashflow valuation (premium/protection legs)
- **Chapter 46** — Intrinsic Spread and Index Basis: Calculating intrinsic spread from constituents
- **Chapter 47** — Index Hedging: Managing index versus single-name hedge mismatches
- **Chapters 48-51** — Tranche Products: CDO tranches built on index portfolios
