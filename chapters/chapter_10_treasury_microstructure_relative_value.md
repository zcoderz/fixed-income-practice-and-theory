# Chapter 10: Treasury Market Microstructure and Relative Value

---

## Introduction

A trader quotes you two five-year Treasury notes with identical maturity dates—one yielding 4.97%, the other 5.02%. They have nearly the same DV01. They pay coupons on the same dates. By any fundamental measure, they should trade at the same yield. Yet one trades five basis points rich to the other, and this spread persists day after day.

What explains this difference? The answer lies not in term structure theory, not in credit risk, and not in tax treatment—but in **market microstructure**: the mechanics of how Treasuries are issued, financed, and traded. The five-basis-point "anomaly" reflects the fact that one bond is *on-the-run* (the most recently issued) while the other is *off-the-run* (an older issue). The on-the-run bond commands a premium because it is more liquid and because holders can finance it at below-market rates in the repo market.

Understanding Treasury microstructure matters for three practical reasons. First, it affects how you measure value: using on-the-run yields as "the" curve can systematically bias your spread calculations. Second, it affects your P&L: the carry on a position depends critically on whether your collateral trades "special" in repo. Third, it creates trading opportunities: relative value strategies that exploit microstructure wedges are a core activity on rates desks.

This chapter examines the "plumbing" that drives these price differences. We begin with the on-the-run phenomenon and the financing channel provided by the repurchase agreement (repo) market. We then analyze how supply and demand for specific collateral creates "special" repo rates, and how these rates drive the wedge between fundamental and observed values. Finally, we integrate these concepts into a relative value framework for trading and curve fitting.

The concepts introduced here connect back to the carry mechanics of Chapter 7 and the repo fundamentals of Chapter 9. They will reappear in Chapter 23 when we examine Treasury futures basis trading.

---

## 10.1 The Treasury Market as an OTC Dealer Market

### Market Structure

The U.S. Treasury market operates as an over-the-counter (OTC) dealer market rather than a centralized exchange. Primary dealers make markets in Treasury securities, posting bid and ask prices to customers and trading among themselves through interdealer brokers. This structure has important implications for liquidity: different securities can have very different bid-ask spreads and market depth, even when their fundamental characteristics are similar.

Treasury prices are quoted in dollars and 32nds per $100 face value. A quote of "99-16" means $99 + 16/32 = 99.50$ per $100 par. Finer increments use fractions of 32nds—"101-04+" or "101-04 5/8" denotes additional precision. This is the **clean price**, which excludes accrued interest; the actual cash exchanged at settlement is the **dirty** (invoice) price.

> **Why microstructure matters:** Tuckman emphasizes that "microstructure" is not merely a technical detail—transaction costs, inventory constraints, and liquidity preferences directly affect traded prices and yields. A bond that appears cheap on a term structure model may be illiquid; a bond that appears rich may command a premium for its financing characteristics. Any relative value analysis must account for these effects.

### Bid-Ask Spreads and Transaction Costs

Bid-ask spreads in Treasuries vary substantially by issue. On-the-run securities typically have the tightest spreads—often 1/64th or less for the most liquid issues. Off-the-run securities trade wider, sometimes several 32nds for illiquid issues. To understand the economic impact of these spreads, consider a bid-ask spread of $1/32$ on $100 million face value. The transaction cost is:

$$\$100{,}000{,}000 \times \frac{1}{32} \times \frac{1}{100} = \$31{,}250$$

This formula reflects that a 1/32 price difference (per $100 face) on $100 million notional translates to $31,250 in P&L. This transaction cost sets a floor on exploitable relative value. Any microstructure "alpha" must exceed trading costs to be economically meaningful.

---

## 10.2 On-the-Run versus Off-the-Run

### Definitions

The **on-the-run (OTR)** Treasury is the most recently issued security in a given maturity sector. It serves as the primary benchmark for that tenor. The **old** issue is the next most recent, the **double-old** is the issue before that, and all older issues are collectively termed **off-the-run**.

As Tuckman explains, "current issues tend to be more liquid. This means that their bid-ask spreads are particularly low and that trades of large size can be conducted relatively quickly."

| Term | Definition |
|------|------------|
| On-the-run (OTR) | Most recently issued Treasury in a maturity sector |
| Old | The previous on-the-run |
| Double-old | The issue before the old |
| Off-the-run | All issues that are not currently on-the-run |

### The Self-Fulfilling Nature of Liquidity

Why do on-the-run issues concentrate liquidity? Tuckman identifies a self-reinforcing dynamic where expectations drive behavior: "Since everyone expects a recent issue to be liquid, investors and traders who demand liquidity and who trade frequently flock to that issue and thus endow it with the anticipated liquidity." Additionally, the dealer community tends to hold substantial inventory of new issues until they "season" and are distributed to buy-and-hold investors. This dealer inventory supports market-making and further enhances liquidity.

The result is a two-tier market: on-the-run securities trade with tight spreads and deep markets, while off-the-run securities—even those with similar maturity and coupon—trade with wider spreads and less depth.

### Why Shorts Prefer On-the-Run

The liquidity advantage of on-the-run securities makes them preferred not only for long positions but also for short positions. Tuckman observes that "most shorts in Treasuries are for relatively brief holding periods: a trading desk hedging the interest rate risk of its current position, a corporation or its underwriter hedging an upcoming sale of its own bonds, or an investor betting that interest rates will rise." Holders of these relatively brief short positions "prefer to sell particularly liquid Treasuries so that, when necessary, they can cover their short positions quickly and at relatively low transaction costs."

This preference creates demand to borrow on-the-run securities, which drives their repo rates lower—a phenomenon we explore next.

> **Analogy: The New Car Smell (Liquidity Premium)**
>
> Why does the OTR bond cost more (yield less)?
> *   **On-the-Run**: The **2025 Model** on the showroom floor. Everyone wants it. Parts are available. It's liquid.
> *   **Off-the-Run**: The **2024 Model**. Just as good functionally (same cash flows), but harder to sell.
> *   **Result**: You pay a premium for the "New" one. This is the **Liquidity Premium**.

---

## 10.3 Repurchase Agreements: The Financing Channel

### Repo as Collateralized Lending

A **repurchase agreement (repo)** is economically equivalent to a collateralized loan, implemented as a sale and repurchase of securities. The borrower sells securities to the lender with an agreement to buy them back at a higher price. The difference between the repurchase price and the original sale price represents the interest on the loan.

For a repo with principal $X$, repo rate $r$, and term $d$ days, the repurchase price is:

$$\boxed{X_{\text{rep}} = X\left(1 + \frac{rd}{360}\right)}$$

The $360$-day denominator reflects standard money market convention (Actual/360 day count).

**Example from Tuckman (Ch 15):** A trading desk finances $\$105,071,823$ overnight at 5.10%. The repurchase amount is:

$$\$105{,}071{,}823 \times \left(1 + \frac{0.0510}{360}\right) = \$105{,}086{,}710$$

The financing cost is $\$14,887$ for one day.

### Reverse Repo and Short Positions

When a trader wants to short a bond they don't own, they must borrow it. The standard mechanism is a **reverse repurchase agreement**: the trader lends cash to a party who owns the bond, taking the bond as collateral. The trader can then deliver the borrowed bond to satisfy their short sale. Tuckman describes this process: "The trading desk finds some party that owns the [bonds]... lends that bank the cash... takes the [bonds] as collateral; and, finally, delivers that collateral to the [buyer]."

The term "reverse repo" emphasizes that the cash lender is motivated by the need to borrow particular bonds, not simply to earn interest on cash.

### Rolling and Covering

A short position must eventually be **covered**—the trader must purchase and deliver the securities they sold and borrowed. If a trader cannot immediately cover, they must **roll** the short by extending the repo or finding another counterparty willing to lend the bonds. The ability to borrow specific securities, and the rate at which one can borrow, is a critical factor in relative value trading.

---

## 10.4 General Collateral versus Special Repo

### The Two-Tier Repo Market

Not all repo is created equal. The market distinguishes between **General Collateral (GC)** and **Special Collateral**. Tuckman explains: "Investors using the repo market to earn interest on cash balances with the security of U.S. Treasury collateral do not usually care about which particular Treasury securities they take as collateral. These investors are said to accept general collateral."

In a GC repo, the cash lender accepts any security from a broad acceptable class (e.g., "any Treasury"). The GC rate reflects the general cost of secured short-term borrowing and typically tracks closely with (but slightly below) the fed funds rate. Tuckman notes that "the GC rate is typically below the fed funds target rate because loans through repurchase agreement are effectively secured by collateral, while loans in the fed funds market are not."

In contrast, a special repo occurs when the cash lender requires a specific security as collateral. The **special rate** for a particular bond depends on supply and demand for that specific issue. When a bond is in high demand for borrowing, its special repo rate falls below the GC rate. The **special spread** measures this difference:

$$\boxed{s \equiv r_{GC} - r_{sp}}$$

A bond "trading special" has $r_{sp} < r_{GC}$, meaning $s > 0$. The owner of a special bond enjoys a financing advantage: they can borrow money at below-GC rates by lending their bond.

### Empirical Magnitudes

Table 10.1 reproduces Tuckman's data on repo rates for February 15, 2001, illustrating the range of special spreads.

**Table 10.1: Selected Repo Rates for February 15, 2001**

| Treasury Issue | Comment | Special Repo Rate |
|---------------|---------|------------------|
| General Collateral | | 5.44% |
| 5.125% 12/31/02 | Old 2-year | 4.75% |
| 4.750% 01/31/03 | On-the-run 2-year | 4.88% |
| 6.750% 05/15/05 | Old 5-year | 5.35% |
| 5.750% 11/15/05 | On-the-run 5-year | 3.85% |
| 6.500% 02/15/10 | Old 10-year | 5.35% |
| 5.750% 08/15/10 | On-the-run 10-year | 4.25% |

*Source: Tuckman, Table 15.1*

The on-the-run five-year traded 159 basis points special (5.44% – 3.85%), while the old five-year traded at essentially GC. As Tuckman notes: "Someone taking the OTR five-year as collateral is willing to lend money overnight at 3.85%, 159 basis points below the GC rate, in order to cover or initiate a short sale."

### Why Bonds Trade Special

The special repo rate is entirely a product of supply and demand for specific collateral. Tuckman identifies several distinct forces that can drive a bond to trade special:

1. **Short interest**: Bonds that many traders want to short become special because shorts must borrow them through reverse repo to make delivery.

2. **Proximity to on-the-run**: Bonds maturing on the same date as an OTR issue may trade somewhat special as substitutes for shorting the OTR. Tuckman notes that "some traders short these issues instead of the most recent issues."

3. **Arbitrage activity**: "Arbitrage traders deciding that a particular sector of the Treasury market is rich relative to swaps might form a large short base... and cause the issues in that sector to trade special."

4. **Inventory effects**: "A large sale of a particular security from the dealer community to an investor that does not participate in the repo market might suddenly make it difficult for shorts to borrow that security."

---

## 10.5 Special Repo Rates and the Auction Cycle

### The Cyclical Pattern

Tuckman documents a systematic pattern in special spreads linked to the Treasury auction cycle: "while the cycle of on-the-run special spreads is far from regular, these spreads tend to be small immediately after auctions and to peak before auctions."

The mechanism is straightforward. Immediately after an auction introduces a new on-the-run security, shorts have a choice between the new OTR and the previous OTR. This substitutability depresses special spreads. Over time, however, the short base coalesces around the new benchmark. As Tuckman explains, "as time passes after an auction, shorts tend to migrate toward the on-the-run security and its special spread tends to rise."

Empirically, Tuckman observes that "special spreads can be quite large: Spreads of 200 to 400 basis points are quite common."

> **Visualization: The Heartbeat of Liquidity**
>
> The Auction Cycle drives the "Heartbeat" of the repo market.
> *   **Auction Day**: huge pulse of liquid supply. Specialness drops. "The patient is flush with blood."
> *   **Weeks Later**: Supply gets locked away in buy-and-hold portfolios. Specialness (scarcity) rises. The market tightens.
> *   **Next Auction**: The "Old" OTR dies (becomes Off-the-Run), and a new Heartbeat starts with the new issue.

### Auction Types and When-Issued Trading

The type of auction influences the magnitude of specialness. A **New Issue** creates a new on-the-run security, pushing the previous OTR to "old" status. A **Reopening**, by contrast, adds supply to the existing on-the-run issue. Tuckman observes that "special spreads for reopened issues do not get as wide as special spreads of new issues," as the additional supply alleviates collateral scarcity.

Bonds can also trade on a **when-issued (WI)** basis before they are actually issued. Tuckman notes that "Bonds to be sold by the Treasury trade on a when-issued basis for a short time before they are actually issued." These WI bonds often trade at a premium to existing OTRs, anticipating the liquidity status they will inherit upon issuance.

---

## 10.6 The Fails Mechanism and the Special Rate Floor

### The Economics of Fails

If a short seller cannot borrow a security to make delivery, they **fail**—they do not deliver the bond and consequently do not receive the sale proceeds. Economically, a fail is equivalent to an involuntary zero-interest loan to the failed-to party: the seller loses one day's interest on the cash proceeds they would have received.

Tuckman explains the implication for special rates. Consider a trader who is short a bond and needs to borrow it through repo:

> "If for some reason the bond cannot be borrowed, the trader will fail to deliver it and, consequently, not receive the proceeds from the sale. In effect, the trader will lose one day of interest on the proceeds. On the other hand, if the bond can be borrowed, the trader will deliver the bond, receive the proceeds, and lend them at the special repo rate. But if the repo rate is 0%, there is no point in bothering with the repo agreement: Earning 0% on the proceeds is the equivalent of having failed to deliver the bond. And certainly the trader will prefer to fail rather than accept a special rate less than 0%."

### The Zero Bound on Special Rates

This logic establishes a historical floor on special rates:

$$\boxed{r_{sp} \geq 0 \quad \Rightarrow \quad s \leq r_{GC}}$$

The special spread cannot exceed the GC rate. As Tuckman notes, "In the fall of 2001, for example, with the GC rate near 2%, the maximum special spread was about 200 basis points."

> **Post-2008 Note:** This analysis reflects the historical environment described in the primary sources. Post-2008 regulatory changes introduced explicit fails charges (often structured as 300 basis points minus the fed funds target, with a floor at zero) to penalize fails more heavily. This mechanism effectively allows for negative repo rates during periods of extreme collateral scarcity. I'm not sure about the precise mechanics of modern fails charges—the sources discuss the zero-floor logic but do not detail the TMPG fails charge framework introduced in 2009.

---

## 10.7 Liquidity Premiums of Recent Issues

### Decomposing the Premium

On-the-run securities trade at higher prices (lower yields) than comparable off-the-run issues. Tuckman decomposes this premium into two distinct components:

> "Some of this premium is due to the demand for shorts and the resulting financing advantage, that is, the ability to borrow money at less than GC rates when using these bonds as collateral. Any additional premium, which might be called a pure liquidity premium, is due to the liquidity demands from long positions."

Market participants often use the term "liquidity premium" to refer to the entire observed yield difference between OTR and off-the-run bonds, encompassing both components.

### Empirical Evidence

Tuckman's data for February 15, 2001 (Table 15.2), shows the OTR two-year trading almost 6 basis points rich to a comparable maturity bond, and the five-year OTR trading 5 basis points rich. The when-issued 10-year traded about 12 basis points rich to the existing OTR.

### Distinguishing Financing Advantage from Pure Liquidity

It is important to understand that financing advantage and pure liquidity have different sources:

> "A pure liquidity premium arises because of a large demand to hold a particular bond relative to the supply of that bond available for trading. A bond trades special because of a large demand to short the bond relative to the supply of bonds available in the repo market."

Since the sources are different, these effects can surface in various permutations. Tuckman provides examples:

- A typical OTR issue, highly valued for liquidity and attracting a large short base, commands both a pure liquidity premium and trades special.
- The 30-year OTR in 2001 was valued for liquidity but few wanted to short it—so it commanded a liquidity premium but did not trade special.
- A seasoned issue with a large short base relative to repo supply might trade special without commanding a liquidity premium.

---

## 10.8 Valuing the Financing Advantage

### Converting Financing Advantage to Yield-Equivalent

A portfolio manager considering two similar bonds must translate the financing advantage into comparable terms. The key is to express the financing benefit in basis points of yield.

**Step 1: Compute the financing advantage in price terms**

For a bond trading at special spread $s$ over horizon $d$ days:

$$\boxed{\text{FinAdv}_{100} = (P + AI) \times s \times \frac{d}{360}}$$

where $P$ is the clean price and $AI$ is accrued interest (both per 100 par).

**Step 2: Convert to yield-equivalent**

Divide by DV01 to express in basis points:

$$\boxed{\text{FinAdv}_{\text{bp}} = \frac{\text{FinAdv}_{100}}{DV01}}$$

### Tuckman's Worked Example

Consider a manager choosing between the OTR five-year (5.75s of 11/15/05) yielding 4.97% and the comparable 5.875s of 11/15/05 yielding 5.02%—a 5 basis point premium for the OTR.

Assume the manager values pure liquidity at 1 basis point, the OTR trades 100 basis points special (on average) over 90 days, and the yield spread narrows from 5 to 3 basis points over the horizon. With a DV01 of 4.261, we can calculate the net advantage following Tuckman's equations (15.16-15.19):

The **financing advantage** is:
$$\frac{0.01 \times 90}{360} = 0.25\% \text{ of face value}$$

The **coupon disadvantage** (since OTR has a coupon 12.5 bp lower) is:
$$\frac{0.00125 \times 90}{365} = 0.0308\% \text{ of face value}$$

The **capital loss from spread narrowing** (2 bp at DV01 4.261) is:
$$2 \times 0.04261\% = 0.0852\% \text{ of face value}$$

The net advantage is:
$$0.25\% - 0.0308\% - 0.0852\% = 0.134\% \text{ of face value}$$

Converting to yield-equivalent: $0.134\% / 0.04261\% \approx 3.1$ basis points.

However, the OTR trades 4 basis points rich (after accounting for 1 bp liquidity value). Tuckman's conclusion: "using the preferences and assumptions of this money manager, the 5.875s are the preferred investment."

---

## 10.9 Benchmark Distortion and Curve Fitting

### The Problem with OTR Benchmarks

On-the-run yields embed liquidity premiums and financing effects that reflect microstructure, not the fundamental term structure. Using OTR yields as "the" curve creates systematic bias.

Tuckman addresses this directly in the context of swap spreads (Ch 18): "While quoted swap spreads are useful for investigating broad themes... in the small these data can be misleading. The problem derives from quoting swap spreads using on-the-run Treasury securities." He explains that "liquidity premiums and special financing have a large impact on the yield of on-the-run government bonds. This means that the swap spread is not a clean measure of the credit of the banking system and of demand and supply in the swap and government bond markets."

### Remedies

Tuckman identifies several approaches to address OTR distortion:

1. **Exclude OTRs** entirely from the curve-fitting universe, using only off-the-run bonds to define the zero curve.

2. **Weight differently**: Give OTRs lower weight in the objective function of the fitting algorithm.

3. **Use multiple curves**: Maintain separate "benchmark" and "off-the-run" curves for different purposes.

4. **Adjust yields explicitly**: Apply explicit liquidity or financing corrections to OTR yields before fitting. Tuckman notes that possible solutions include "measuring swap spreads with respect to fitted yields from an off-the-run government curve or explicitly adjusting on-the-run yield for financing and liquidity."

Tuckman observes that "both of these solutions require a good deal of subjective judgement" and therefore "adjusted swap spreads tend to be used by individual trading desks and houses rather than being widely quoted in the marketplace."

### Swap Spread Measurement

The conventional swap spread—swap rate minus Treasury yield—is particularly affected by benchmark choice. A swap spread computed against the OTR five-year may differ by several basis points from one computed against a fitted curve yield.

**Example:** If the 5-year swap rate is 5.20% and the OTR yield is 4.80%, the headline spread is 40 bp. But if the fitted off-the-run yield is 4.85%, the "fundamental" spread is only 35 bp. This 5 basis point difference is economically significant relative to typical bid-ask spreads.

---

## 10.10 Application: The September 2001 Collateral Crisis

### Market Stress Reveals Microstructure

The terrorist attacks of September 11, 2001, created unprecedented disruption in the Treasury repo market. As Tuckman documents: "The terrorist attack on the World Trade Center disrupted the specials market in two ways. First, the resulting confusion and the destruction of records caused many government bond transactions to fail... Second, heightened uncertainty and credit concerns caused many participants in the repo markets to pull their securities from the repo market."

The result was a severe shortage of on-the-run collateral. Table 10.2 shows average repo rates on September 20, 2001.

**Table 10.2: Selected Repo Rates on September 20, 2001**

| Treasury Issue | Comment | Special Repo Rate |
|---------------|---------|------------------|
| General Collateral | | 1.75% |
| 3.625% 08/31/03 | On-the-run 2-year | 0.65% |
| 4.625% 05/15/06 | On-the-run 5-year | 0.10% |
| 5.000% 08/15/11 | On-the-run 10-year | 0.35% |

*Source: Tuckman, Table 15.3*

Tuckman notes: "the GC rate was 1.75% while the fed funds rate at the time was 3%. The widespread shortage of Treasury collateral widened the spread between fed funds and GC to 125 basis points." The five-year OTR was trading at just 0.10%—essentially at the fails floor.

### The Treasury's Response

In an unprecedented move on October 4, 2001, the Treasury announced a surprise auction of $6 billion of the OTR 10-year note. As Tuckman emphasizes: "This was unprecedented in two ways. First, the Treasury usually keeps to a strict issuance calendar. Second, the Treasury almost always gives much more notice of auctions. These policies are designed to foster market stability, but on October 4 the Treasury judged that drastic action was required."

The market impact was immediate. The 10-year futures fell, the spread between the OTR and old 10-year tightened from 5.2 to 3.7 basis points, and the OTR special spread collapsed by approximately 100 basis points.

This episode illustrates how microstructure effects can dominate during stress: collateral scarcity drove large price and spread moves independent of fundamental changes in the risk-free rate.

> **Narrative: The Repo Squeeze**
>
> 1.  **Setup**: Too many shorts crowd into the OTR 10y (betting on rates rising).
> 2.  **Trigger**: Market panic (e.g., 9/11). Lenders pull collateral back to safety.
> 3.  **Consequence**: Shorts *must* borrow the bond to deliver, but no one is lending.
> 4.  **Price**: Repo rate hits 0% (or -3% today). Shorts bleed cash daily to hold the position.
> 5.  **Blowout**: The price of the OTR *spikes* relative to the curve. Shorts are forced to cover at any price, driving it higher.

---

## 10.11 Relative Value Framework

### Decomposing Observed Spreads

For any comparison between Treasury issues, the observed yield or price difference can be decomposed into three parts:

$$\boxed{\text{Observed diff} = \underbrace{\text{Curve component}}_{\text{fundamental}} + \underbrace{\text{Financing component}}_{\text{specialness}} + \underbrace{\text{Pure liquidity}}_{\text{benchmark demand}}}$$

Only the first component is "fundamental" in the sense of reflecting the term structure of discount rates. The latter two are microstructure effects that can persist, vary over time, and create trading opportunities.

### Implications for Hedging

Hedging with on-the-run securities introduces exposure to microstructure shocks. If you hedge a corporate bond position using the OTR Treasury, you are not just hedging duration risk; you are acquiring exposure to:

- Changes in the OTR liquidity premium
- Changes in the OTR special spread
- Roll effects when a new issue becomes OTR

These exposures can cause P&L variation even when "rates" (in the fundamental sense) are stable.

### RV Signals and Microstructure

Relative value signals based on OTR benchmarks require careful interpretation:

- OTR/off-run yield spreads often proxy for liquidity conditions rather than stable fair value
- Swap spreads using OTR may move for microstructure reasons unrelated to credit or swap market supply
- Auction-cycle positioning matters because specialness follows a predictable pattern

As Tuckman emphasizes: "Any trade or hedge involving a security that is trading special requires the same set of assumptions and calculations... How much is liquidity worth? How will special spreads behave? How will the premium change over time?"

> **Visualization: The RV Trade (The Fly Chart)**
>
> *   **X-axis**: Maturity.
> *   **Y-axis**: Yield - Fitted Curve (Spread to Curve).
> *   **The Visual**: Most dots scatter around 0. The OTR points are typically **below** the line (Rich/Expensive).
> *   **The Trade**: "Buy the cheap dot, Sell the rich dot" (Convergence Arbitrage).
> *   **Warning**: The "Coupon Effect" often distorts raw yields. You must compare "Apples to Apples" (using a Fitted Curve or Z-Spread) to strip out structural bias.
> *   **Famous Failure**: LTCM (1998) went Long Off-the-Run / Short On-the-Run. A "Flight to Quality" blew out the spread, and they couldn't hold on for convergence.

---

## 10.12 Carry and P&L Decomposition with Financing

### The Carry Formula

Following Tuckman's notation (Ch 15, Eq 15.8), the P&L from holding a bond over $d$ days can be decomposed into price change and carry:

$$\boxed{P\&L = \underbrace{P(d) - P(0)}_{\text{Price change}} + \underbrace{\frac{cd}{D} - (P(0) + AI(0)) \cdot \frac{rd}{360}}_{\text{Carry}}}$$

where $P(0), P(d)$ are clean prices, $c$ is the annual coupon rate, $D$ is days in the coupon period, and $r$ is the repo rate.

Carry is simply interest income minus financing cost. If the bond trades special ($r = r_{sp} < r_{GC}$), the financing cost is lower, and carry is higher than for a GC-financed position.

### Breakeven Analysis

Carry is useful for computing breakeven price changes. Tuckman provides an example: "an investor might plan to purchase the 5⅞s of November 15, 2005, for an invoice price of 105.103073 and hold them for 30 days. If the 30-day term rate for financing the bonds is 5.10%, how big a price decline can occur before the investment shows a loss?"

With carry of approximately $40,190 on $100 million face, the price can fall by about 4 cents per 100 face value before the trade turns unprofitable. Specialness directly affects this breakeven: if the bond goes special, carry improves and the breakeven cushion widens.

---

## 10.13 Forward Prices and the Repo Connection

### No-Arbitrage Forward Price

For a bond with no coupon payment during the forward window, the forward price is tied to spot price and repo by no-arbitrage. Tuckman derives this in Chapter 16 (Eq 16.7):

$$\boxed{P_{fwd} + AI(d) = (P(0) + AI(0))\left(1 + \frac{rd}{360}\right)}$$

Rearranging (following Eq 16.8):

$$\boxed{P_{fwd} = P(0) - \text{Carry}}$$

The forward price equals the spot price minus carry. If carry is positive (income exceeds financing), the forward clean price is below spot. If carry is negative, the forward price is above spot.

### The Repo Rate in Forward Pricing

The repo rate $r$ in these formulas is the rate at which *that specific bond* can be financed. For a special bond, $r = r_{sp} < r_{GC}$, so carry is higher and the forward price is lower than for an otherwise identical bond financing at GC.

This means spot versus forward yield spreads can differ substantially: even if two bonds have similar spot yields, differences in expected specialness create different forward yields. Tuckman illustrates this with the OTR and old five-year: "the forward yield spread was only 3.4 basis points, 8.5 basis points below the 11.9 spot yield spread... The market priced the bonds... under the assumption that the OTR and old five-year would not then trade so differently in the special repo market."

---

## Summary

1.  **Market structure matters:** The Treasury market is an OTC dealer market where liquidity concentrates in on-the-run issues. Off-the-run bonds, even with similar fundamentals, trade differently.

2.  **Repo is the financing backbone:** Repurchase agreements allow dealers to finance inventory and traders to borrow securities for short positions. The repo rate directly affects carry and position economics.

3.  **Special versus GC:** On-the-run securities typically trade "special"—their repo rates are below GC because of strong demand to borrow them. Special spreads can be 100-400 basis points.

4.  **Auction cycle dynamics:** Special spreads are small after auctions and tend to widen toward the next auction as the short base migrates to the OTR.

5.  **Liquidity premium decomposition:** OTR richness reflects both cheaper financing (borrowing availability) and pure liquidity demand (benchmark status).

6.  **Fails create a floor:** Historically, special rates cannot go below 0% because failing to deliver is equivalent to financing at 0%. This bounds the special spread at the GC rate.

7.  **Benchmark distortion:** Using OTR yields as "the curve" systematically biases spread measurements. Practitioners must use fitted curves or adjust for liquidity effects to isolate fundamental value.

8.  **Convert financing to yield-equivalent:** Divide the financing advantage (in price terms) by DV01 to express in basis points for apples-to-apples comparison.

9.  **Stress reveals microstructure:** The September 2001 episode showed how collateral scarcity can drive extreme special spreads and price dislocations even without fundamental rate changes.

10. **RV decomposition:** Observed yield differences = curve component + financing component + pure liquidity. Only the first is "fundamental."

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| On-the-run (OTR) | Most recently issued Treasury of a maturity | Concentrates liquidity and short interest |
| Special repo | Repo rate below GC for a specific bond | Affects carry and relative value |
| Special spread | $s = r_{GC} - r_{sp}$ | Measures financing advantage |
| Liquidity premium | OTR yield depression vs comparable off-run | Sum of financing + pure liquidity effects |
| Financing advantage | $(P + AI) \times s \times d/360$ | Dollar benefit of trading special |
| Yield-equivalent | FinAdv / DV01 | Converts to comparable basis points |
| Fails | Non-delivery of sold securities | Creates 0% floor (historically) on special rates |
| Auction cycle | Special spreads small after auctions, rise before | Creates predictable RV patterns |

---

## Notation for This Chapter

| Symbol | Definition |
|--------|------------|
| $r_{GC}$ | General collateral repo rate |
| $r_{sp}$ | Special repo rate |
| $s = r_{GC} - r_{sp}$ | Special spread |
| $d$ | Holding period (days) |
| $P$ | Clean price per 100 par |
| $AI$ | Accrued interest per 100 par |
| $DV01$ | Price change per bp yield change (per 100) |
| $P_{fwd}$ | Forward clean price |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is "on-the-run" in Treasuries? | The most recently issued Treasury of a maturity sector. |
| 2 | Why do OTR issues concentrate liquidity? | It is self-fulfilling: traders expecting liquidity flock to OTRs, endowing them with the anticipated liquidity. |
| 3 | Define repo in one sentence. | A collateralized loan implemented as a sale and repurchase of securities. |
| 4 | What is the repo interest formula? | Repurchase price $= X(1 + rd/360)$. |
| 5 | What is GC repo? | Repo where any Treasury is acceptable as collateral. |
| 6 | What is special repo? | Repo requiring a specific security; the rate is often below GC. |
| 7 | Define special spread. | $s = r_{GC} - r_{sp}$. |
| 8 | Why might a bond trade special? | High demand to borrow it for short positions. |
| 9 | What creates the OTR liquidity premium? | Two components: financing advantage + pure liquidity demand. |
| 10 | How do you convert financing advantage to bp? | Divide price-point advantage by DV01. |
| 11 | What bounds the special spread? | The GC rate (since special cannot go below 0% in the historical model). |
| 12 | Why couldn't special rates go negative historically? | Failing to deliver is equivalent to 0% financing, so no one would lend at negative rates. |
| 13 | What is the auction-cycle pattern for specials? | Small after auctions, rising toward next auction. |
| 14 | How do reopenings affect specials? | They add supply, so specials don't get as wide as with new issues. |
| 15 | What is when-issued trading? | Trading bonds before they are actually issued. |
| 16 | Why can OTR yields distort curve fitting? | They embed liquidity premiums beyond the fundamental term structure. |
| 17 | How to remedy OTR distortion? | Exclude OTRs, weight them less, or explicitly adjust yields. |
| 18 | What is a reverse repo? | Lending cash to borrow specific securities. |
| 19 | What is "covering a short"? | Purchasing and delivering securities previously sold and borrowed. |
| 20 | What happened to specials in September 2001? | Extreme shortages drove rates near zero; Treasury made a surprise auction to alleviate the squeeze. |

---

## Mini Problem Set

**1.** A 5-year Treasury has DV01 = 0.045 per 100. If yields fall 6 bp, estimate the price change.

**Solution:** $\Delta P \approx -DV01 \times \Delta y = -0.045 \times (-6) = +0.27$ points per 100.

---

**2.** You finance $\$200$ million overnight at 4.80%. Compute interest cost (ACT/360).

**Solution:** Cost $= 200{,}000{,}000 \times 0.048 \times (1/360) = \$26{,}667$.

---

**3.** A bond trades 300 bp special (GC = 5.00%, special = 2.00%). For $\$100$mm face over 7 days, compute financing advantage.

**Solution:** FinAdv $= 100{,}000{,}000 \times 0.03 \times (7/360) = \$58{,}333$.

---

**4.** Using Problem 3, if DV01 = $\$85{,}000$ per bp, what is the yield-equivalent advantage?

**Solution:** $58{,}333 / 85{,}000 \approx 0.69$ bp.

---

**5.** Swap rate = 5.15%, OTR 5-year yield = 4.75%, fitted curve yield = 4.80%. What are the two swap spreads?

**Solution:** OTR spread = 40 bp; fitted spread = 35 bp; difference = 5 bp.

---

**6.** A short must borrow a bond at special 0.25% when GC is 4.75%, for 14 days on $\$50$mm. Compute incremental cost vs GC.

**Solution:** Cost $= 50{,}000{,}000 \times 0.045 \times (14/360) = \$87{,}500$.

---

**7.** Using the forward-price formula: $P(0) = 100.00$, $AI(0) = 0.50$, $P_{fwd} = 100.10$, $AI(d) = 0.80$, $d = 30$. Solve for implied repo.

**Solution:**
- $(P_{fwd} + AI(d)) / (P(0) + AI(0)) = 100.90 / 100.50 = 1.00398$
- $r = (360/30) \times 0.00398 = 4.78\%$

---

**8.** Explain qualitatively why OTR/off-run yield spreads might widen during market stress even if the fitted curve slope is unchanged.

**Solution Sketch:** During stress, "flight to quality" drives demand for the most liquid assets (OTR), pushing their yields down relative to off-the-runs. Simultaneously, collateral scarcity may drive OTRs to trade special, further lowering their yields via the financing component. These are microstructure effects, not term structure slope changes.

---

**9.** After an auction, would you expect the new OTR to be more or less special than immediately before? Why?

**Solution Sketch:** Less special (or not special at all initially). The new supply and the presence of the old OTR as a substitute for shorts means supply is ample relative to demand. Specialness typically builds up over time as the issue seasons and shorts accumulate.

---

**10.** Design a trade to profit from an expected widening of the OTR five-year special spread over the next month. What are the risks?

**Solution Sketch:** Buy the OTR 5-year and short the old 5-year (or a matched maturity substitute). As the OTR becomes more special, its relative financing advantage grows, potentially widening the price spread. Risks include the "pure" yield curve changing (if not duration matched perfectly) or the OTR losing specialness faster than expected (e.g., if a reopening is announced).

---

**11.** How would you adjust a swap spread calculation to remove OTR distortion?

**Solution Sketch:** Instead of $r_{swap} - y_{OTR}$, use $r_{swap} - y_{fitted}$, where $y_{fitted}$ is the yield implied by a curve fitted to off-the-run bonds.

---

**12.** If a bond's special spread widens from 50 to 150 bp over 30 days, how does this affect carry for a long position?

**Solution Sketch:** It improves carry. The owner of the bond can lend it out at the lower special rate ($r_{sp}$ decreases), effectively borrowing cash at a cheaper rate. This reduces financing costs and increases net carry.

---

## Source Map

### (A) Verified Facts (Source-Backed)

| Fact | Source |
|------|--------|
| On-the-run vs off-the-run definitions | Tuckman Ch 15 |
| Self-fulfilling nature of OTR liquidity | Tuckman Ch 15 |
| Repo repurchase price formula | Tuckman Ch 15, Eq 15.3-15.4 |
| GC vs special repo definitions | Tuckman Ch 15 |
| GC rate typically below fed funds (secured vs unsecured) | Tuckman Ch 15 |
| Special spread magnitude (159 bp for 5-year) | Tuckman Table 15.1 |
| Shorts prefer liquid securities | Tuckman Ch 15 |
| Auction-cycle pattern for specials | Tuckman Ch 15 |
| Reopenings produce smaller special spreads | Tuckman Ch 15 |
| Fails mechanism and 0% floor | Tuckman Ch 15 |
| Liquidity premium decomposition (financing + pure liquidity) | Tuckman Ch 15 |
| Liquidity premium magnitudes (5-6 bp for 5-year) | Tuckman Table 15.2 |
| Financing advantage calculation | Tuckman Ch 15, Eq 15.16-15.19 |
| September 2001 collateral crisis data | Tuckman Ch 15, Table 15.3 |
| Treasury surprise auction response | Tuckman Ch 15 |
| Carry decomposition formula | Tuckman Ch 15, Eq 15.8 |
| Forward price = Spot - Carry | Tuckman Ch 16, Eq 16.7-16.8 |
| OTR yields distort swap spread measurement | Tuckman Ch 18 |
| Solutions: fitted curves or adjusted yields | Tuckman Ch 18 |
| Forward yield spread vs spot yield spread difference | Tuckman Ch 18 |

### (B) Reasoned Inference (Derived from A)

| Inference | Derivation Logic |
|-----------|------------------|
| RV decomposition: curve + financing + liquidity | Combines liquidity premium definition with carry mechanics |
| Hedging with OTR creates microstructure exposure | Follows from OTR yield including non-fundamental components |
| Time-varying liquidity premium | Implied by auction-cycle patterns and crisis behavior |
| Special spread bounded by GC rate | Direct implication of fails/zero-bound logic |
| Forward price lower when special | Follows from $P_{fwd} = P - \text{Carry}$ where carry is higher when special |

### (C) Flagged Uncertainties

| Item | Uncertainty |
|------|-------------|
| Exact current U.S. Treasury auction calendar | Not enumerated in provided sources |
| Modern fails charges and penalty mechanics | Not detailed beyond the zero-floor logic (TMPG charges post-2009 are not in Tuckman 2nd Ed) |
| Precise empirical magnitude of when-issued cheapening | Would require specific empirical study |
| Open repo mechanics (indefinite term, daily rate reset) | Sources discuss overnight and term repo but do not define open repo specifically |
