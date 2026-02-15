# Chapter 10: Treasury Market Microstructure and Relative Value

---

## Introduction

A trader quotes you two five-year Treasury notes with identical maturity dates—one yielding 4.97%, the other 5.02%. They have nearly the same DV01. They pay coupons on the same dates. By any fundamental measure, they should trade at the same yield. Yet one trades five basis points rich to the other, and this spread persists day after day.

What explains this difference? The answer lies not in term structure theory, not in credit risk, and not in tax treatment—but in **market microstructure**: the mechanics of how Treasuries are issued, financed, and traded. The five-basis-point "anomaly" reflects the fact that one bond is *on-the-run* (the most recently issued) while the other is *off-the-run* (an older issue). The on-the-run bond commands a premium because it is more liquid and because holders can finance it at below-market rates in the repo market.

Understanding Treasury microstructure matters for three practical reasons. First, it affects how you measure value: using on-the-run yields as "the" curve can systematically bias your spread calculations. Second, it affects your P&L: the carry on a position depends critically on whether your collateral trades "special" in repo. Third, it creates trading opportunities: relative value strategies that exploit microstructure wedges are a core activity on rates desks.

This chapter also examines two watershed events that revealed the fragility of convergence arbitrage: the **LTCM collapse of 1998**, where the "flight to quality" proved that apparently riskless trades can face unlimited mark-to-market losses, and the **March 2020 "dash for cash"**, where even Treasuries—the world's ultimate safe asset—experienced severe dislocations. These case studies illustrate why microstructure is not merely academic: it can determine whether a desk survives a crisis.

We begin with the on-the-run phenomenon and the primary dealer system that intermediates Treasury markets. We examine the financing channel provided by the repurchase agreement (repo) market, analyzing how supply and demand for specific collateral creates "special" repo rates. We then integrate these concepts into a relative value framework for trading and curve fitting, including quantitative liquidity measures and butterfly trade mechanics.

Prerequisites: [Chapter 7 — Bond Return Decomposition](chapters/chapter_07_bond_return_decomposition.md), [Chapter 9 — Repo](chapters/chapter_09_repo_funding_engine.md), [Chapter 11 — DV01/PV01](chapters/chapter_11_dv01_pv01_definitions_computation.md)  
Follow-on: [Chapter 23 — Treasury Futures](chapters/chapter_23_treasury_futures.md), [Chapter 28 — Basis Trades](chapters/chapter_28_basis_trades.md)

## Learning Objectives
- After this chapter, you can explain why on-the-run Treasuries can trade rich to off-the-run issues (liquidity + financing specialness).
- You can translate a Treasury quote (price/yield + repo) into carry and forward-price intuition.
- You can compute a financing advantage/drag in dollars and convert it to a yield-equivalent using a stated DV01 bump object, units, and sign.
- You can design and sanity-check a DV01-neutral relative value trade (e.g., OTR vs old issue; butterfly) and list the key failure modes.

---

## 10.1 The Treasury Market as an OTC Dealer Market

### 10.1.1 Market Structure

The U.S. Treasury market operates as an over-the-counter (OTC) dealer market rather than a centralized exchange. Primary dealers make markets in Treasury securities, posting bid and ask prices to customers and trading among themselves through interdealer brokers. This structure has important implications for liquidity: different securities can have very different bid-ask spreads and market depth, even when their fundamental characteristics are similar.

Treasury prices are commonly quoted in dollars and 32nds per $100 face value. A quote of `99-16` means \(99 + 16/32 = 99.5\) per $100 par. A `+` conventionally denotes an extra \(1/64\) (a half-tick). These are **clean prices**, excluding accrued interest. The cash exchanged at settlement is based on the **dirty (invoice) price**:

$$\boxed{P_{\text{dirty}} = P_{\text{clean}} + AI}$$

### 10.1.2 The Primary Dealer System

The U.S. Treasury market is intermediated by a network of **primary dealers**—securities firms that are counterparties of the Federal Reserve Bank of New York in its market operations and that are expected to participate meaningfully in Treasury auctions and make markets in U.S. government securities. The roster changes over time; for operational questions, use the New York Fed’s primary dealer list as the source of truth.

> **Desk Reality:** Dealer intermediation is balance-sheet constrained.  
> **Common break:** In stress, liquidity can deteriorate quickly even in Treasuries as dealers become less willing/able to intermediate large flows.  
> **What to check:** For any RV trade, stress-test (i) wider bid/ask and lower depth, and (ii) changes in specialness/financing, not just yield convergence.

### 10.1.3 Bid-Ask Spreads and Transaction Costs

Bid-ask spreads in Treasuries vary by issue and market conditions. To understand the economic impact of spreads, consider a bid-ask spread of \(1/32\) on $100 million face value. The round-trip spread cost (crossing once) is:

$$\$100{,}000{,}000 \times \frac{1}{32} \times \frac{1}{100} = \$31{,}250$$

This formula reflects that a 1/32 price difference (per $100 face) on $100 million notional translates to $31,250 in P&L. This transaction cost sets a floor on exploitable relative value. Any microstructure "alpha" must exceed trading costs to be economically meaningful.

### 10.1.4 Measuring Liquidity Quantitatively

While bid-ask spreads are directly observable in quoted markets, quantifying liquidity for less actively traded securities often requires inference. One classic approach is **Roll’s implicit spread model**, which infers an effective bid-ask spread from the negative first-order autocovariance of transaction price changes.

Let \(X_t = P_t - P_{t-1}\) be the transaction price change. In the simple Roll (1984) bid-ask “bounce” model (often written for returns/log-prices), the first-lag autocovariance is negative and satisfies \(\mathrm{Cov}(X_t, X_{t-1})=-S^2/4\), implying:

$$\boxed{S = 2\sqrt{-\mathrm{Cov}(X_t, X_{t-1})}}$$

This estimator is most useful as a *relative* liquidity measure (e.g., comparing issues or regimes) because real markets include inventory effects, price impact, and other microstructure features beyond bid-ask bounce.

> **Caveat:** Roll’s model is highly stylized. It can be distorted by price impact, discrete pricing, and news-driven order flow, so treat it as a rough proxy rather than a literal quoted spread.

---

## 10.2 On-the-Run versus Off-the-Run

### 10.2.1 Definitions

The **on-the-run (OTR)** Treasury is the most recently issued security in a given maturity sector. It serves as the primary benchmark for that tenor. The **old** issue is the next most recent, the **double-old** is the issue before that, and all older issues are collectively termed **off-the-run**.

| Term | Definition |
|------|------------|
| On-the-run (OTR) | Most recently issued Treasury in a maturity sector |
| Old | The previous on-the-run |
| Double-old | The issue before the old |
| Off-the-run | All issues that are not currently on-the-run |

### 10.2.2 The Self-Fulfilling Nature of Liquidity

Why do on-the-run issues concentrate liquidity? Part of the answer is self-reinforcing: because many participants *expect* the benchmark issue to be easiest to trade, they route their activity there, which in turn makes it the easiest to trade. Dealer intermediation and inventory distribution further reinforce this concentration.

The result is a two-tier market: on-the-run securities trade with tight spreads and deep markets, while off-the-run securities—even those with similar maturity and coupon—trade with wider spreads and less depth.

### 10.2.3 The OTR Lifecycle

The on-the-run “status” is not permanent. A simplified lifecycle is:

1. **When-issued (WI)** trading: the soon-to-be-issued bond can trade on a forward-settlement basis before it is issued.
2. **On-the-run phase**: after auction/issuance, the new issue becomes the benchmark for its tenor and trading activity concentrates there.
3. **Roll to old/off-the-run**: when a newer benchmark is issued, the previous benchmark becomes “old,” and liquidity and specialness can fade as the issue ages.

This lifecycle helps explain why liquidity premia and repo specialness can vary systematically over time even when the bond’s contractual cashflows have not changed.

### 10.2.4 Why Shorts Prefer On-the-Run

The liquidity advantage of on-the-run securities makes them preferred not only for long positions but also for short positions: it is operationally easier to cover (buy back) a short in a liquid benchmark issue than in a less-liquid off-the-run.

This preference creates demand to borrow on-the-run securities, which drives their repo rates lower—a phenomenon we explore next.

> **Analogy: The New Car Smell (Liquidity Premium)**
>
> Why does the OTR bond cost more (yield less)?
> *   **On-the-Run**: The **2025 Model** on the showroom floor. Everyone wants it. Parts are available. It's liquid.
> *   **Off-the-Run**: The **2024 Model**. Just as good functionally (same cash flows), but harder to sell.
> *   **Result**: You pay a premium for the "New" one. This is the **Liquidity Premium**.

---

## 10.3 Repurchase Agreements: The Financing Channel

### 10.3.1 Repo as Collateralized Lending

A **repurchase agreement (repo)** is economically equivalent to a collateralized loan, implemented as a sale and repurchase of securities. The borrower sells securities to the lender with an agreement to buy them back at a higher price. The difference between the repurchase price and the original sale price represents the interest on the loan.

For a repo with principal $X$, repo rate $r$, and term $d$ days, the repurchase price is:

$$\boxed{X_{\text{rep}} = X\left(1 + \frac{rd}{360}\right)}$$

The \(360\)-day denominator reflects standard money-market convention (Actual/360 day count) used in many repo calculations.

### 10.3.2 Reverse Repo and Short Positions

When a trader wants to short a bond they don’t own, they must borrow it. One common mechanism is a **reverse repurchase agreement (reverse repo)**: the trader lends cash and receives the security as collateral, which can then be delivered to settle the short sale.

The term "reverse repo" emphasizes that the cash lender is motivated by the need to borrow particular bonds, not simply to earn interest on cash.

### 10.3.3 Rolling and Covering

A short position must eventually be **covered**—the trader must purchase and deliver the securities they sold and borrowed. If a trader cannot immediately cover, they must **roll** the short by extending the repo or finding another counterparty willing to lend the bonds. The ability to borrow specific securities, and the rate at which one can borrow, is a critical factor in relative value trading.

---

## 10.4 General Collateral versus Special Repo

### 10.4.1 The Two-Tier Repo Market

Not all repo is created equal. The market distinguishes between **general collateral (GC)** and **special collateral**. In a **GC repo**, the cash lender accepts *any* security from an acceptable class (e.g., “any Treasury”). In a **special repo**, the trade specifies a particular security because that issue is scarce or in demand (for example, because it is cheapest-to-deliver or heavily shorted).

When a bond is in high demand to borrow, its **special repo rate** \(r_{sp}\) can fall below the **GC repo rate** \(r_{GC}\). The **special spread** measures this wedge:

$$\boxed{s \equiv r_{GC} - r_{sp}}$$

A bond “trading special” has \(r_{sp} < r_{GC}\) and therefore \(s>0\). Intuitively, the bond owner enjoys a financing advantage (they can finance the bond more cheaply), while a short position in the bond faces a financing drag (it must source the bond in a tight collateral market).

### 10.4.2 Empirical Magnitudes

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

In this snapshot, the on-the-run five-year traded 159 basis points special (5.44% – 3.85%), while the old five-year traded near GC. Interpreting this in desk terms: shorts were willing to accept a below-GC rate on cash in order to borrow the benchmark security, and owners of the security enjoyed unusually cheap financing.

### 10.4.3 Why Bonds Trade Special

Special repo rates are a product of supply and demand for *specific* collateral. Common drivers include:

1. **Short interest / deliverability**: Heavy shorting (or a need to deliver into another contract) increases demand to borrow the issue.

2. **Close substitutes**: Nearby issues (sometimes even with the same maturity date) can become “substitute shorts” and share some specialness.

3. **Arbitrage positioning**: RV trades that require shorting a sector can create a short base that tightens collateral availability.

4. **“Locked up” supply**: When an issue is held by investors who do not lend it, the lendable float shrinks and specials can widen.

---

## 10.5 Special Repo Rates and the Auction Cycle

### 10.5.1 The Cyclical Pattern

Specialness often has a “seasoning” pattern around the auction cycle. A useful stylized fact (documented in Tuckman’s discussion of specials) is:

- **Special spreads tend to be smaller soon after auctions** and can become larger as time passes and a short base develops.
- **Reopenings** (which add supply to an existing issue) tend to keep specials from becoming as wide as they can become for **new issues**.

One intuition is substitutability: immediately after a new issue is auctioned, shorts can borrow either the new benchmark or the previous benchmark more easily. Over time, activity concentrates into the new benchmark and specialness can build.

> **Visualization: The Heartbeat of Liquidity**
>
> The Auction Cycle drives the "Heartbeat" of the repo market.
> *   **Auction Day**: huge pulse of liquid supply. Specialness drops. "The patient is flush with blood."
> *   **Weeks Later**: Supply gets locked away in buy-and-hold portfolios. Specialness (scarcity) rises. The market tightens.
> *   **Next Auction**: The "Old" OTR dies (becomes Off-the-Run), and a new Heartbeat starts with the new issue.

### 10.5.2 Auction Types and When-Issued Trading

A **new issue** creates a new on-the-run security and pushes the previous benchmark to “old.” A **reopening** adds supply to an existing issue (often the current benchmark), which can alleviate collateral scarcity and dampen specialness.

Treasury securities can also trade on a **when-issued (WI)** basis shortly before they are issued. Conceptually, WI trading is forward-settlement trading that helps price discovery for the soon-to-be benchmark security.

---

## 10.6 The Fails Mechanism and the Special Rate Floor

### 10.6.1 The Economics of Fails

Consider a trader who is short an on-the-run Treasury and needs to borrow it through a repurchase agreement to make delivery. If the bond cannot be borrowed, the trader **fails** to deliver it and consequently does not receive the proceeds from the sale; in effect, the trader loses one day of interest on the proceeds.

This creates a “borrow versus fail” comparison for a short position:
- **Borrow** the bond: you can deliver, receive the short-sale proceeds, and lend those proceeds at the bond’s special repo rate.
- **Fail**: you do not receive proceeds on time and can be worse off than borrowing, depending on the settlement/fails convention in force.

### 10.6.2 The Zero Bound on Special Rates (Historical)

In the simplified setting (no explicit fails charge), borrowing at a very low special rate can become economically similar to failing. This motivates a stylized “floor” intuition:

$$\boxed{r_{sp} \geq 0 \quad \Rightarrow \quad s \leq r_{GC}}$$

Intuition: if the special repo rate were \(0\%\), earning \(0\%\) on the short-sale proceeds is economically similar to having failed to deliver. A trader would prefer to fail rather than accept a special rate below \(0\%\). Therefore, in this stylized setting, the special rate cannot fall below \(0\%\), and equivalently the special spread cannot exceed the GC rate.

This is not a law of nature: it is an economic comparison that changes when settlement conventions (including explicit fails charges) change.

### 10.6.3 Modern Fails Charges (Post-2009)

Modern Treasury settlement practice includes an explicit daily fails charge under the Treasury Market Practices Group (TMPG) recommended “fails charge trading practice.” For delivery-versus-payment (DVP) Treasury trades, TMPG specifies a **daily dollar charge** that depends on trade proceeds and a reference rate:

$$\boxed{C_{\text{daily}} \;=\; \frac{1}{360}\times 0.01 \times \max(3 - R,\;F)\times P_{\text{proceeds}}}$$

where:
- \(C_{\text{daily}}\) is the fails charge amount in dollars per day,
- \(P_{\text{proceeds}}\) is the trade proceeds (the funds due from the non-failing party in a DVP settlement),
- \(R\) is the TMPG reference rate (defined from the fed funds target; if a target range applies, TMPG uses the lower limit),
- \(F\) is a floor (in percentage points) that depends on the TMPG rule version and effective date.

> **Desk Reality:** “Borrow vs fail” is a real financing decision.  
> **Common break:** Backtests and RV screens often assume GC financing, but the actual economics can flip when a security is special or hard-to-borrow.  
> **What to check:** For any short, compare the all-in expected cost of borrowing (special rate + balance-sheet/operational frictions) to the effective cost of failing under the applicable settlement convention.

Because failing now carries an explicit penalty, the “effective bound” for how special a security can become can be lower than 0% in practice. The exact economics depend on the applicable TMPG parameters and settlement mechanics, so treat any “floor” as a model assumption, not a universal constant.

---

## 10.7 Liquidity Premiums of Recent Issues

### 10.7.1 Decomposing the Premium

On-the-run securities often trade at higher prices (lower yields) than comparable off-the-run issues. A useful decomposition is:

1. **Financing component:** the value of being able to finance the benchmark cheaply (special repo).
2. **Pure liquidity component:** the residual value investors place on holding the benchmark for ease of trading (even after financing is accounted for).

In practice, market participants may refer to the total wedge as a “liquidity premium,” even though part of it can be traced to financing.

### 10.7.2 Empirical Evidence

Tuckman’s snapshot for settlement on February 15, 2001 (Table 15.2) illustrates the idea: the on-the-run two-year and five-year traded several basis points rich to comparable-maturity issues, and the when-issued 10-year traded at an even larger premium relative to the existing on-the-run.

### 10.7.3 Distinguishing Financing Advantage from Pure Liquidity

Financing advantage and pure liquidity come from different “flows”:
- **Financing (specialness)** is driven by the demand to *borrow* a specific issue (often for shorts and deliverability).
- **Pure liquidity** is driven by the demand to *hold/trade* the benchmark issue as a liquid instrument.

Because these drivers are distinct, it is possible to see combinations like:

- A typical OTR issue, highly valued for liquidity and attracting a large short base, commands both a pure liquidity premium and trades special.
- The 30-year OTR in 2001 was valued for liquidity but few wanted to short it—so it commanded a liquidity premium but did not trade special.
- A seasoned issue with a large short base relative to repo supply might trade special without commanding a liquidity premium.

---

## 10.8 Valuing the Financing Advantage

### 10.8.1 Converting Financing Advantage to Yield-Equivalent

A portfolio manager considering two similar bonds must translate the financing advantage into comparable terms. The key is to express the financing benefit in basis points of yield.

**Step 1: Compute the financing advantage in price terms**

For a bond trading at special spread \(s=r_{GC}-r_{sp}\) over a holding horizon of \(d\) days, a convenient *per-100-face* approximation is:

$$\boxed{\text{FinAdv}_{100} = (P + AI) \times s \times \frac{d}{360}}$$

where $P$ is the clean price and $AI$ is accrued interest (both per 100 par).

**Step 2: Convert to yield-equivalent**

Define the (yield-bumped) DV01 used in this chapter as:

$$\boxed{DV01_y := P(y-1\text{ bp}) - P(y)}$$

where the **bump object** is the bond’s quoted yield \(y\), and \(DV01_y\) is measured in **price points per 100 face per 1bp**. With this convention, a long position has \(DV01_y>0\) and the first-order approximation is:

$$\Delta P \approx -DV01_y \cdot \Delta y_{\text{bp}}$$

Then the financing advantage can be expressed in yield-equivalent basis points by dividing by \(DV01_y\):

$$\boxed{\text{FinAdv}_{\text{bp}} = \frac{\text{FinAdv}_{100}}{DV01_y}}$$

The same calculation can be read as a **financing drag** for a short position in the special bond: being short forces you to source the bond in the specials market, which can materially reduce (or even dominate) the expected convergence P&L.

### 10.8.2 Tuckman's Worked Example

Tuckman frames the “is the benchmark worth it?” question as a comparison between:
- the **yield premium** you pay for the on-the-run issue, and
- your (subjective) value of liquidity plus the **expected financing advantage** from special repo.

The key practical move is converting financing advantage into a yield-equivalent (bp) using a stated DV01 convention, so the terms are comparable.

---

## 10.9 Benchmark Distortion and Curve Fitting

### 10.9.1 The Problem with OTR Benchmarks

On-the-run yields embed liquidity premiums and financing effects that reflect microstructure, not the fundamental term structure. Using OTR yields as "the" curve creates systematic bias.

Tuckman emphasizes this point in the context of swap spreads: if your “Treasury leg” is an on-the-run benchmark, the measured spread can move because the benchmark’s liquidity/specialness moves, even if the underlying curve or swap market has not changed.

### 10.9.2 Remedies

Tuckman identifies several approaches to address OTR distortion:

1. **Exclude OTRs** entirely from the curve-fitting universe, using only off-the-run bonds to define the zero curve.

2. **Weight differently**: Give OTRs lower weight in the objective function of the fitting algorithm.

3. **Use multiple curves**: Maintain separate "benchmark" and "off-the-run" curves for different purposes.

4. **Adjust yields explicitly**: Apply explicit liquidity or financing corrections to OTR yields before fitting (a desk-by-desk approach that depends on model choices and judgement).

### 10.9.3 Swap Spread Measurement

The conventional swap spread—swap rate minus Treasury yield—is particularly affected by benchmark choice. A swap spread computed against the OTR five-year may differ by several basis points from one computed against a fitted curve yield.

**Example:** If the 5-year swap rate is 5.20% and the OTR yield is 4.80%, the headline spread is 40 bp. But if the fitted off-the-run yield is 4.85%, the "fundamental" spread is only 35 bp. This 5 basis point difference is economically significant relative to typical bid-ask spreads.

---

## 10.10 Application: The September 2001 Collateral Crisis

### 10.10.1 Market Stress Reveals Microstructure

Tuckman describes how the September 11, 2001 shock disrupted the specials market through settlement disruption (fails) and a pullback of collateral lending. The result was a severe shortage of on-the-run collateral. Table 10.2 shows average repo rates on September 20, 2001.

**Table 10.2: Selected Repo Rates on September 20, 2001**

| Treasury Issue | Comment | Special Repo Rate |
|---------------|---------|------------------|
| General Collateral | | 1.75% |
| 3.625% 08/31/03 | On-the-run 2-year | 0.65% |
| 4.625% 05/15/06 | On-the-run 5-year | 0.10% |
| 5.000% 08/15/11 | On-the-run 10-year | 0.35% |

*Source: Tuckman, Table 15.3*

In this snapshot, the five-year on-the-run traded at a very low repo rate (highly special), consistent with acute collateral scarcity and “borrow vs fail” economics.

### 10.10.2 The Treasury's Response

Tuckman notes that the Treasury responded with an unusual surprise auction announcement, which helped alleviate scarcity in benchmark collateral and changed specials-market dynamics.

This episode illustrates how microstructure effects can dominate during stress: collateral scarcity drove large price and spread moves independent of fundamental changes in the risk-free rate.

> **Narrative: The Repo Squeeze**
>
> 1.  **Setup**: Too many shorts crowd into the OTR 10y (betting on rates rising).
> 2.  **Trigger**: Market panic (e.g., 9/11). Lenders pull collateral back to safety.
> 3.  **Consequence**: Shorts *must* borrow the bond to deliver, but no one is lending.
> 4.  **Price**: The special repo rate can fall toward 0%. Under explicit fails-charge regimes, the effective “specialness bound” can extend below 0%. Shorts bleed cash daily to keep the position on.
> 5.  **Blowout**: The price of the OTR *spikes* relative to the curve. Shorts are forced to cover at any price, driving it higher.

---

## 10.11 Case Study: LTCM and the Limits of Convergence Arbitrage (1998)

### 10.11.1 The Trade (Convergence Arbitrage)

A canonical Treasury convergence trade is:
- **Long** a cheaper, less-liquid off-the-run issue.
- **Short** a richer, more-liquid on-the-run issue.
- Size the legs to be approximately **DV01-neutral**, so the trade is primarily exposed to changes in the *liquidity/financing wedge*, not the level of rates.

The economic bet is that the on-the-run premium (liquidity + financing) will mean-revert as the benchmark ages and is replaced by a new benchmark.

### 10.11.2 Why Leverage Shows Up

When the mispricing is only a few basis points, investors often use leverage to make the trade’s expected return economically meaningful. The cost is that the position becomes fragile: spread widening can trigger margin calls and forced deleveraging.

### 10.11.3 1998: Flight to Quality and a Widening of Liquidity Spreads

In August 1998, Russia defaulted on its debt and this led to what is termed a “flight to quality” in capital markets: investors valued liquid instruments more highly than usual and the spreads between the prices of liquid and illiquid instruments increased dramatically. In Hull’s summary of the episode, “the prices of the bonds [LTCM] had bought went down and the prices of those it had shorted increased.” For a Treasury convergence trade (long off-the-run, short on-the-run), that is the same painful pattern.

Hull also emphasizes a feedback loop: when many participants run similar convergence trades, attempts to liquidate simultaneously can further widen liquidity spreads.

### 10.11.4 Lessons for Relative Value Trading

1. **“DV01-neutral” is not “riskless.”** You are still exposed to liquidity and financing shocks.
2. **Leverage turns small wedges into large P&L swings.** Margining and funding constraints can force liquidation before convergence.
3. **Crowding matters.** If others hold the same trade, unwind dynamics can dominate fundamentals.

> **Desk Reality:** Convergence requires survival.  
> **Common break:** A trade that “should converge eventually” can blow up on the path if spreads widen and financing tightens.  
> **What to check:** Stress-test a widening in the on-the-run liquidity premium and an increase in specialness (financing drag), not just parallel yield shocks.

---

## 10.12 Case Study: March 2020 and the Dash for Cash

### 10.12.1 The Setup: Dealer Balance Sheet Constraints

In late February and March 2020, volatility and illiquidity rose sharply across markets. The Federal Reserve’s June 2020 Monetary Policy Report describes heavy selling pressure in Treasuries from a range of investors and notes that dealers approached balance-sheet constraints, limiting their ability to intermediate.

### 10.12.2 The "Dash for Cash"

Unlike 1998's flight *to* quality, March 2020 saw a flight *from* everything—including Treasuries—into cash. The mechanism:

1. **Margin calls**: Falling equity and credit prices triggered margin calls across portfolios.
2. **Cash demand**: Investors needed to raise cash immediately, selling whatever was liquid.
3. **Treasury selling**: Paradoxically, Treasuries—the most liquid asset—were sold heavily precisely because they could be sold.
4. **Limited intermediation capacity**: Dealers were less able to absorb the flow at the speed/size demanded.

### 10.12.3 Market Dysfunction

Market functioning deteriorated: bid-ask spreads widened, market depth declined, and relative value relationships (including on-the-run/off-the-run wedges) became unstable.

### 10.12.4 Federal Reserve Intervention

The Monetary Policy Report highlights two core stabilizers:
1. **Large-scale Treasury purchases** to absorb selling pressure.
2. **Repo operations** to support funding and market functioning.

These actions contributed to improving Treasury market functioning as conditions stabilized.

### 10.12.5 Lessons for Modern Markets

March 2020 revealed new fragilities:

1. **Liquidity can evaporate in benchmark markets.** “Risk-free” does not mean “always liquid.”
2. **Intermediation capacity is finite.** When flows are one-sided and large, RV relationships can gap.
3. **Benchmarks can move for microstructure reasons.** Hedging with an on-the-run instrument can embed liquidity/specialness exposure.

> **Desk Reality:** Stress-testing “microstructure risk” is part of RV.  
> **Common break:** A DV01-neutral trade looks safe in a curve model but blows up when funding/liquidity wedges widen.  
> **What to check:** Scenario analysis for (i) wider specials, (ii) wider bid/ask and lower depth, and (iii) forced unwind/margining dynamics.

---

## 10.13 Relative Value Framework

### 10.13.1 Decomposing Observed Spreads

For any comparison between Treasury issues, the observed yield or price difference can be decomposed into three parts:

$$\boxed{\text{Observed diff} = \underbrace{\text{Curve component}}_{\text{fundamental}} + \underbrace{\text{Financing component}}_{\text{specialness}} + \underbrace{\text{Pure liquidity}}_{\text{benchmark demand}}}$$

Only the first component is "fundamental" in the sense of reflecting the term structure of discount rates. The latter two are microstructure effects that can persist, vary over time, and create trading opportunities.

### 10.13.2 Implications for Hedging

Hedging with on-the-run securities introduces exposure to microstructure shocks. If you hedge a corporate bond position using the OTR Treasury, you are not just hedging duration risk; you are acquiring exposure to:

- Changes in the OTR liquidity premium
- Changes in the OTR special spread
- Roll effects when a new issue becomes OTR

These exposures can cause P&L variation even when "rates" (in the fundamental sense) are stable.

### 10.13.3 RV Signals and Microstructure

Relative value signals based on OTR benchmarks require careful interpretation:

- OTR/off-run yield spreads often proxy for liquidity conditions rather than stable fair value
- Swap spreads using OTR may move for microstructure reasons unrelated to credit or swap market supply
- Auction-cycle positioning matters because specialness follows a predictable pattern

Trades and hedges involving special collateral require explicit assumptions: how much liquidity is worth to you, how special spreads may evolve, and how quickly you need the trade to converge.

---

## 10.14 Carry and P&L Decomposition with Financing

### 10.14.1 The Carry Formula

Following Tuckman's notation (Ch 15, Eq 15.8), the P&L from holding a bond over $d$ days can be decomposed into price change and carry:

$$\boxed{P\&L = \underbrace{P(d) - P(0)}_{\text{Price change}} + \underbrace{\frac{cd}{D} - (P(0) + AI(0)) \cdot \frac{rd}{360}}_{\text{Carry}}}$$

where $P(0), P(d)$ are clean prices, $c$ is the annual coupon rate, $D$ is days in the coupon period, and $r$ is the repo rate.

Carry is simply interest income minus financing cost. If the bond trades special ($r = r_{sp} < r_{GC}$), the financing cost is lower, and carry is higher than for a GC-financed position.

### 10.14.2 Breakeven Analysis

Carry gives a quick breakeven rule of thumb. Ignoring second-order effects, your holding-period P&L is:

$$P\&L \approx \Delta P_{\text{clean}} + \text{Carry}$$

So if carry is positive, the **clean price can fall by roughly “carry (in price points)” before the trade turns unprofitable**. Because special repo lowers the financing cost term, a bond that becomes more special typically has better carry and therefore a larger breakeven cushion (all else equal).

---

## 10.15 Forward Prices and the Repo Connection

### 10.15.1 No-Arbitrage Forward Price

For a bond with no coupon payment during the forward window, the forward price is tied to spot price and repo by no-arbitrage. Tuckman derives this in Chapter 16 (Eq 16.7):

$$\boxed{P_{fwd} + AI(d) = (P(0) + AI(0))\left(1 + \frac{rd}{360}\right)}$$

Rearranging (following Eq 16.8):

$$\boxed{P_{fwd} = P(0) - \text{Carry}}$$

The forward price equals the spot price minus carry. If carry is positive (income exceeds financing), the forward clean price is below spot. If carry is negative, the forward price is above spot.

### 10.15.2 The Repo Rate in Forward Pricing

The repo rate $r$ in these formulas is the rate at which *that specific bond* can be financed. For a special bond, $r = r_{sp} < r_{GC}$, so carry is higher and the forward price is lower than for an otherwise identical bond financing at GC.

This means spot versus forward comparisons can look very different once financing is included. Even if two bonds have similar spot yields today, differences in *expected* specialness can imply different forward prices and forward yields.

---

## 10.16 Practical Relative Value Analysis

### 10.16.1 The Rich/Cheap Framework

Relative value analysis begins with comparing observed yields to a fitted curve. The **deviation from fitted curve** measures how cheap or rich a bond is:

$$\text{RV Score} = y_{\text{observed}} - y_{\text{fitted}}$$

Positive = cheap (higher yield than model); negative = rich (lower yield than model).

**The visual representation:**

- **X-axis**: Maturity
- **Y-axis**: Yield spread to fitted curve
- **Points**: Individual bonds
- **Pattern**: OTR points typically below zero (rich); off-the-run scattered around zero

This "fly chart" is the standard visualization for Treasury relative value.

### 10.16.2 Butterfly Trades

A **butterfly trade** is a three-leg position designed to isolate **curvature** (the belly versus the wings) rather than the level of rates.

**The 2-5-10 Butterfly:**

- **Short** the 2-year (front wing)
- **Long** the 5-year (body)
- **Short** the 10-year (back wing)

The weights are typically chosen to be **DV01-neutral** (net DV01 \(\approx 0\)). Some desks also target variants like cash-neutral or price-neutral, but DV01-neutral is the first sanity check.

If $DV01_2$, $DV01_5$, $DV01_{10}$ are the DV01s, the hedge ratios solve:

$$\alpha \cdot DV01_2 + DV01_5 + \beta \cdot DV01_{10} = 0$$

With one degree of freedom, a common convention is to allocate **equal DV01 to each wing** (so each wing offsets half the body DV01). With the body notionals normalized to \(+1\):

$$\boxed{\alpha = -\frac{DV01_5}{2 \cdot DV01_2}, \qquad \beta = -\frac{DV01_5}{2 \cdot DV01_{10}}}$$

In words: short each wing in an amount that contributes half of the body DV01. The fly profits when the body outperforms (its yield falls relative to a weighted average of the wings) and loses when it underperforms.

### 10.16.3 The "Spread of Spreads" (Cheapness Indicator)

The **spread of spreads** compares how different maturity sectors are valued relative to their fitted curve positions:

$$\text{SoS}_{5y} = (\text{RV Score of 5y OTR}) - (\text{RV Score of 5y off-run})$$

A large negative SoS indicates the OTR is extremely rich relative to comparable off-the-run bonds—a potential convergence trade if you believe the spread will normalize.

### 10.16.4 Asset Swap Spread Analysis

The **asset swap spread (ASW)** is another relative value lens for comparing bonds to the swap curve. Conceptually, it is the spread \(s_{ASW}\) such that discounting the bond’s cashflows using swap discount rates **plus that spread** reproduces the bond price.

Two practical cautions:
1. **ASW is not generally equal to “bond yield minus swap rate.”** That equality only holds in special cases (e.g., very restrictive curve shapes/assumptions).
2. For Treasuries, ASW (and related “spread to swaps” measures) can still move for microstructure reasons (special financing, benchmark demand), not just “fundamental” curve movements.

### 10.16.5 Putting It Together: A Rich/Cheap Trade

**Scenario:** The 5-year OTR trades 8 bp rich to the fitted curve, while the old 5-year trades 2 bp cheap. The spread is 10 bp wider than average.

**Trade:**
- Long old 5-year
- Short OTR 5-year
- Duration-matched

**Expected P&L drivers:**
1. **Spread convergence**: If the 10 bp reverts to 5 bp average, gain 5 bp × DV01
2. **Financing**: The short OTR will cost financing (it trades special); the long position may finance at GC
3. **Roll**: When the next auction makes the OTR "old," its liquidity premium should decline

**Risks:**
- Flight to quality widens OTR premium (LTCM scenario)
- Special spread increases, making the short more expensive
- Curve moves adversely if the duration match is imperfect

> **Desk Reality: Execution Matters**
>
> Rich/cheap trades often look attractive on paper but face execution challenges. Shorting the OTR means borrowing it at special rates. If the bond is 150 bp special, you're paying 150 bp/year in carry drag. This cost must be factored into expected returns. Many trades that look profitable ignoring financing are actually losers after financing.

> **Pitfall — Ignoring financing in “convergence” trades:** Treating an OTR/off-the-run spread as “pure value” and forgetting that the short leg may be expensive to borrow.  
> **Why it matters:** Financing drag can overwhelm the P&L from spread convergence.  
> **Quick check:** Compute \(\text{breakeven bp} \approx \text{(financing drag in \$)} / \text{(trade DV01 in \$/bp)}\).

### 10.16.6 Worked Example: DV01-Neutral OTR vs Old 5-Year (Financing Drag Breakeven)

**Context**
- You believe the on-the-run 5-year is rich versus a close substitute and will cheapen (converge) over the next two months.
- You want a trade whose primary exposure is to the *relative* move (microstructure wedge), not the level of rates.

**Timeline (Make Dates Concrete)**
- Trade date: 2026-02-17  
- Settlement date: 2026-02-18 (assume T+1 for Treasuries)  
- Exit date: 2026-04-19 (60 days holding period)  
- Coupon payments during holding: assume none (for simplicity)

**Inputs**
- Old 5-year (long leg):
  - Dirty price \(P_{\text{dirty}}\) \(\approx 100.50\) per 100
  - Repo rate \(r_{GC} = 4.80\\%\\) (ACT/360)
  - \(DV01_{y,\text{old}} = 0.044\) price points per 100 per 1bp
- On-the-run 5-year (short leg):
  - Dirty price \(P_{\text{dirty}}\) \(\approx 100.50\) per 100
  - Special repo rate \(r_{sp} = 3.30\\%\\) (ACT/360)
  - \(DV01_{y,\text{OTR}} = 0.045\) price points per 100 per 1bp
- Special spread: \(s = r_{GC} - r_{sp} = 1.50\\%\\)
- Notional: long \(N_{\text{old}} = \\$100\\text{mm}\) face. Choose short notional to be DV01-neutral:
  $$N_{\text{OTR}} = N_{\text{old}}\\cdot \\frac{DV01_{y,\text{old}}}{DV01_{y,\text{OTR}}} \\approx 100\\text{mm}\\cdot \\frac{0.044}{0.045} \\approx \\$97.8\\text{mm}$$

**Outputs (What You Produce)**
- Trade DV01 to parallel yield shifts \(\approx 0\) by construction.
- Financing drag from shorting a special bond (in dollars).
- Breakeven convergence (in bp) needed to pay for financing drag.

**Step-by-step**
1. **Compute trade DV01 (dollars per bp).**  
   Long-leg DV01:
   $$DV01_{\\$,\\text{old}} = 0.044\\times \\frac{100{,}000{,}000}{100} = \\$44{,}000\\;\\text{per bp}$$
   Short-leg DV01:
   $$DV01_{\\$,\\text{OTR}} = 0.045\\times \\frac{97{,}800{,}000}{100} \\approx \\$44{,}010\\;\\text{per bp}$$
   so the net DV01 is approximately zero.

2. **Compute financing drag (short OTR).**  
   The short position must source the on-the-run security in the specials market. A rough financing-drag estimate over \(d\) days is:
   $$\\text{Drag} \\approx \\left(\\frac{N_{\\text{OTR}}}{100}\\cdot P_{\\text{dirty}}\\right)\\cdot s\\cdot \\frac{d}{360}$$
   Plugging in \(N_{\\text{OTR}}\\approx 97.8\\text{mm}\), \(P_{\\text{dirty}}\\approx 100.50\), \(s=1.50\\%\\), \(d=60\):
   $$\\text{Drag} \\approx \\left(97.8\\text{mm}\\times \\frac{100.50}{100}\\right)\\times 0.015\\times \\frac{60}{360} \\approx \\$246{,}000$$

3. **Breakeven convergence.**  
   If the on-the-run cheapens by \(\Delta y\) bp versus the old issue (e.g., \(y_{OTR}-y_{old}\) increases toward 0), the convergence P&L is on the order of:
   $$\\text{P\\&L}_{\\text{conv}} \\approx DV01_{\\$,\\text{OTR}}\\cdot \\Delta y$$
   Breakeven \(\Delta y\) is therefore:
   $$\\Delta y_{\\text{breakeven}} \\approx \\frac{\\$246{,}000}{\\$44{,}000} \\approx 5.6\\text{ bp}$$

**Cashflows (table)**
| Date | Cashflow | Explanation |
|---|---:|---|
| 2026-02-18 → 2026-04-19 | \(-\\$246{,}000\\) | Financing drag from being short a special bond for 60 days (rough estimate) |
| Exit | \(+DV01\\times \\Delta y\) | Convergence P&L depends on how much the OTR cheapens vs the old issue |

**P&L / Risk Interpretation**
- This trade is *not* a bet on parallel rates: it is a bet that the on-the-run premium will shrink.
- Financing is a first-order driver: if specialness widens while you wait, your expected P&L can flip sign.
- A DV01-neutral construction does **not** neutralize liquidity premium risk, specialness risk, or funding/margin dynamics.

**Sanity Checks**
- Units check: \((\\$\\times \\%)\\times (\\text{days}/360)\\) produces dollars.
- Sign check: larger \(s\) (more special) increases drag for the short; convergence must be larger/faster to compensate.
- Limit check: if \(s=0\), financing drag vanishes and the trade reduces to a pure relative-yield bet.

---

## Summary

1.  **Market structure matters:** The Treasury market is an OTC dealer market where liquidity concentrates in on-the-run issues. Off-the-run bonds, even with similar fundamentals, trade differently.

2.  **Repo is the financing backbone:** Repurchase agreements allow dealers to finance inventory and traders to borrow securities for short positions. The repo rate directly affects carry and position economics.

3.  **Special versus GC:** On-the-run securities often trade "special"—their repo rates are below GC because of strong demand to borrow them. Special spreads can become large enough that financing dominates short-horizon P&L.

4.  **Auction cycle dynamics:** Special spreads are small after auctions and tend to widen toward the next auction as the short base migrates to the OTR.

5.  **Liquidity premium decomposition:** OTR richness reflects both cheaper financing (borrowing availability) and pure liquidity demand (benchmark status).

6.  **Fails change the economics:** In the absence of an explicit fails charge, failing can act like a “0% alternative” that limits how negative special repo can go. With TMPG-style fails charges, the borrow-versus-fail comparison can permit negative specials in practice.

7.  **Benchmark distortion:** Using OTR yields as "the curve" systematically biases spread measurements. Practitioners must use fitted curves or adjust for liquidity effects.

8.  **Convert financing to yield-equivalent:** Translate a dollar financing advantage/drag into yield-equivalent bp by dividing by \(DV01_{\$}\) (or equivalently divide the per-100 financing advantage by \(DV01_y\)).

9.  **Convergence arbitrage has risks:** LTCM (1998) demonstrated that "riskless" convergence trades can incur massive losses when flight to quality widens OTR premiums.

10. **Stress can overwhelm even Treasuries:** March 2020 illustrated that in a dash-for-cash episode, aggressive selling pressure can widen bid-ask spreads and dislocate on-the-run/off-the-run relationships as intermediation capacity is strained.

11. **Butterflies isolate curvature:** The 2-5-10 fly trades body vs. wings, profiting from curvature changes rather than level.

12. **Roll's model quantifies illiquidity:** Negative first-order autocovariance in transaction-price changes implies an effective bid-ask spread: \(S = 2\sqrt{-\mathrm{Cov}(X_t, X_{t-1})}\).

---

## Key Concepts

| Concept | Definition | Why It Matters |
|---|---|---|
| Clean vs dirty price | Clean excludes accrued; \(P_{\text{dirty}}=P_{\text{clean}}+AI\) | Carry/financing and cash settlement use invoice (dirty) amounts |
| On-the-run (OTR) | Most recently issued Treasury in a maturity sector | Benchmark liquidity + deliverability; often trades rich |
| Off-the-run | Older, non-benchmark issues | Wider spreads; can appear “cheap” relative to fitted curves |
| Repo | Sale + forward repurchase (collateralized financing) | Links cash price, forward price, and carry |
| GC repo | Repo where collateral is a broad Treasury set | Baseline funding rate for Treasury collateral |
| Special repo | Repo requiring a specific security | Collateral scarcity shows up as below-GC rates |
| Special spread | \(s := r_{GC}-r_{sp}\) (annualized, ACT/360) | Converts financing wedges into dollars/bp; \(s>0\) means “special” |
| Financing advantage / drag | \(\approx (N/100)\,P_{\text{dirty}}\,s\,d/360\) dollars over \(d\) days (advantage if long collateral; drag if short) | Often first-order in RV P&L over short horizons |
| \(DV01_y\) (price DV01) | \(DV01_y := P(y-1\text{bp})-P(y)\) (price points per 100 per bp) | Locks bump object/sign for yield-equivalent comparisons |
| \(DV01_{\$}\) (dollar DV01) | \(DV01_{\$} := DV01_y\times N/100\) (dollars per bp) | Position risk scalar; used to size DV01-neutral trades |
| TMPG fails charge | \(C_{\text{daily}}=(1/360)\,0.01\,\max(3-R,F)\,P_{\text{proceeds}}\) (dollars/day) | Makes failing explicitly costly; affects the “negative special” bound |
| Liquidity premium decomposition | OTR richness = financing component + pure liquidity component | Prevents double-counting when fitting curves / valuing benchmarks |
| Roll implied spread | \(S = 2\sqrt{-\mathrm{Cov}(X_t, X_{t-1})}\) | Infers a rough effective spread from bid-ask bounce |
| Butterfly (2-5-10) | 3-leg trade (short wings, long body), sized to net DV01 \(\approx 0\) | Isolates curvature from level; common RV building block |
| Asset swap spread (ASW) | Spread \(s_{ASW}\) that, when applied to swap discounting/float, makes PV match the bond price | Another rich/cheap lens vs swaps; sensitive to benchmark choice |

---

## Notation

| Symbol | Meaning | Units / Convention |
|---|---|---|
| \(P_{\text{clean}}\) | Clean price | price points per 100 par; quoted in 32nds |
| \(AI\) | Accrued interest | price points per 100 par |
| \(P_{\text{dirty}}=P_{\text{clean}}+AI\) | Dirty (invoice) price | used for cash settlement and financing approximations |
| \(N\) | Face value (notional) | dollars of par (e.g., \(\$100\)mm) |
| \(r_{GC}\) | GC repo rate | annualized simple rate; ACT/360; decimal (0.05 = 5%) |
| \(r_{sp}\) | Special repo rate (security-specific) | same convention as \(r_{GC}\) |
| \(s=r_{GC}-r_{sp}\) | Special spread | decimal per year; \(s>0\Rightarrow\) “special” |
| \(d\) | Holding period | calendar days; ACT/360 |
| \(P_{fwd}\) | Forward clean price | price points per 100 at forward settlement |
| \(P_{\text{proceeds}}\) | Trade proceeds in a DVP settlement | dollars |
| \(C_{\text{daily}}\) | TMPG fails charge | dollars per day; \(0.01\max(3-R,F)\) uses percent points |
| \(R\) | TMPG reference rate | percent per year (not decimal); based on fed funds target |
| \(F\) | TMPG floor parameter | percent points; depends on rule version/date |
| \(y\) | Bond yield quote | bump object for \(DV01_y\) |
| \(DV01_y\) | Price DV01 to yield bump | price points per 100 per 1bp; \(DV01_y=P(y-1\text{bp})-P(y)>0\) for long |
| \(DV01_{\$}\) | Dollar DV01 of a position | dollars per bp; \(DV01_{\$}=DV01_y\times N/100\) |
| \(X_t\) | Transaction price change | price points per 100; \(X_t=P_t-P_{t-1}\) |
| \(S\) | Roll implied spread | price points per 100 |

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
| 10 | How do you convert financing advantage to bp? | Divide a dollar financing advantage/drag by \(DV01_{\$}\) (or per-100 financing advantage by \(DV01_y\)). |
| 11 | What historically bounded the special spread? | The GC rate (since special could not go below 0% before 2009). |
| 12 | Why couldn't special rates go negative historically? | Failing to deliver was equivalent to 0% financing, so no one would lend at negative rates. |
| 13 | What is the TMPG fails charge formula (DVP Treasuries)? | \(C_{\text{daily}}=(1/360)\,0.01\,\max(3-R,F)\,P_{\text{proceeds}}\) dollars/day. |
| 14 | How do fails charges allow negative repo rates? | They create a penalty for failing that exceeds the cost of negative repo. |
| 15 | What is the auction-cycle pattern for specials? | Small after auctions, rising toward next auction. |
| 16 | How do reopenings affect specials? | They add supply, so specials don't get as wide as with new issues. |
| 17 | What is when-issued trading? | Trading bonds before they are actually issued. |
| 18 | What is the dirty (invoice) price? | The cash-settlement price: \(P_{\text{dirty}}=P_{\text{clean}}+AI\) per 100 par. |
| 19 | Why can OTR yields distort curve fitting? | They embed liquidity premiums beyond the fundamental term structure. |
| 20 | How to remedy OTR distortion? | Exclude OTRs, weight them less, or explicitly adjust yields. |
| 21 | What is Roll's implicit spread formula? | \(S = 2\sqrt{-\mathrm{Cov}(X_t, X_{t-1})}\), where \(X_t=P_t-P_{t-1}\). |
| 22 | What does LTCM stand for and what was their strategy? | Long-Term Capital Management; convergence arbitrage (long cheap/illiquid, short rich/liquid). |
| 23 | What triggered LTCM's collapse? | Russia's default in August 1998 caused a "flight to quality" that widened spreads against their positions. |
| 24 | What DV01 convention is used in this chapter? | \(DV01_y:=P(y-1\text{bp})-P(y)\) (per 100); \(DV01_{\$}=DV01_y\times N/100\); \(\Delta P\approx -DV01_y\Delta y_{\text{bp}}\). |
| 25 | What was the "dash for cash" in March 2020? | Investors sold even Treasuries to raise cash, causing unprecedented dislocations. |
| 26 | Why can even OTR liquidity deteriorate in stress? | Intermediation capacity is finite: risk limits, balance-sheet usage, and funding frictions can force dealers to widen spreads and reduce size. |
| 27 | What is a butterfly trade? | A three-leg trade (short wings, long body) that isolates curvature from level. |
| 28 | What is the "spread of spreads"? | The difference between OTR and off-run richness scores—a measure of relative liquidity premium. |
| 29 | What are primary dealers? | Securities firms designated by the Fed to participate in auctions and make markets in Treasuries. |
| 30 | What is an asset swap spread? | The spread \(s_{ASW}\) that makes the PV of bond cashflows discounted off the swap curve (with the spread applied per the convention) equal the bond’s price. |

---

## Mini Problem Set

**1.** A 5-year Treasury has \(DV01_y = 0.045\) (price points per 100 per bp). If yields fall 6 bp, estimate the price change.

**Solution:** Using \(DV01_y=0.045\) (points per 100 per bp) and \(\Delta P \approx -DV01_y\Delta y_{\text{bp}}\): \(\Delta P \approx -0.045\times(-6)=+0.27\) points per 100.

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

**Solution Sketch:** Less special initially. The new supply and the presence of the old OTR as a substitute for shorts means supply is ample relative to demand. Specialness typically builds up over time as the issue seasons and shorts accumulate.

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

**13.** Using Roll's model, you compute \(\mathrm{Cov}(X_t, X_{t-1}) = -0.0001\), where \(X_t=P_t-P_{t-1}\) and prices are in dollars per 100. Estimate the bid-ask spread.

**Solution:**
$$S = 2\sqrt{-(-0.0001)} = 2\sqrt{0.0001} = 2 \times 0.01 = 0.02$$
The implied spread is 2 cents per $100 face, or about 0.6/32nds.

---

**14.** LTCM was long off-the-run bonds and short on-the-run bonds. Explain why "flight to quality" hurt this position.

**Solution Sketch:** Flight to quality drives demand for the most liquid, safest assets—on-the-run Treasuries. This pushes OTR prices up (yields down), widening the OTR premium over off-the-runs. LTCM was short the asset that everyone wanted to buy and long the asset that everyone was avoiding. Spread widening caused mark-to-market losses; leverage magnified these into catastrophic losses.

---

**15.** In March 2020, even on-the-run Treasuries experienced wide bid-ask spreads. Give two market-structure reasons why bid-ask spreads can widen sharply in stress.

**Solution Sketch:** In stress, (i) dealers hit risk limits / balance-sheet usage constraints, (ii) funding and margin terms worsen, and (iii) hedging becomes more expensive and less reliable. The result is less willingness to warehouse inventory and wider bid-ask spreads.

---

**16.** A 2-5-10 butterfly has \(DV01_{y,2} = 0.018\), \(DV01_{y,5} = 0.042\), \(DV01_{y,10} = 0.078\) (price points per 100 per bp). You want \(\$100\)mm notional in the body. Calculate the wing sizes for a DV01-neutral fly.

**Solution Sketch:**
- Body DV01 = $100mm \times 0.00042 = \$42{,}000$
- If we weight wings equally: each wing should contribute $\$21{,}000$ of DV01
- 2-year notional: $21{,}000 / 0.00018 = \$116.7mm$
- 10-year notional: $21{,}000 / 0.00078 = \$26.9mm$

The fly is: short $116.7mm 2-year, long $100mm 5-year, short $26.9mm 10-year.

---

**17.** A DVP Treasury trade has proceeds \(P_{\text{proceeds}}=\$50{,}000{,}000\). The TMPG reference rate is \(R=0.25\%\). Assume \(F=0\). Compute the TMPG daily fails charge \(C_{\text{daily}}\).

**Solution:** With \(\max(3-R,F)=3-0.25=2.75\) (percent points):
\[
C_{\text{daily}}=\frac{1}{360}\times 0.01\times 2.75\times 50{,}000{,}000 \approx \$3{,}819\;\text{per day.}
\]
Qualitatively: an explicit fails charge makes failing costly, so borrowing at a negative special rate can be cheaper than failing in some scenarios.

---

**18.** You're analyzing a rich/cheap trade: OTR 5y trades 7 bp rich to curve; old 5y trades 1 bp cheap. You are short \(\$100\)mm face of the OTR and long \(\$100\)mm face of the old (assume prices near par), so the package has \(DV01_{\$}\approx \$45{,}000\) per bp. The OTR is 150 bp special (old is at GC). Holding period is 90 days. If the spread halves (to 4 bp), calculate P&L including financing.

**Solution Sketch:**
- **Spread convergence P&L**: 4 bp × $45,000 = $180,000 gain
- **Financing drag of short OTR** (150 bp special): \(0.015\times 100{,}000{,}000\times 90/360 \approx \$375{,}000\)
- **Net P&L**: $180,000 - $375,000 = **-$195,000 loss**

The financing drag exceeds the spread convergence gain. This illustrates why financing costs matter.

---

## References
- (Bruce Tuckman, *Fixed Income Securities*, “Special Repo Rates and the Auction Cycle”; “Liquidity Premiums of Recent Issues”; “Valuing the Financing Advantage”; “Asset Swap Spreads and Asset Swaps”)
- (Salih N. Neftci, *Principles of Financial Engineering*, “Repurchase Agreements”; “Special Versus General Collateral”)
- (Robert A. Jarrow, *Modeling Fixed Income Securities and Interest Rate Options*, “Treasury Security Markets”)
- (John C. Hull, *Risk Management and Financial Institutions*, “Long-Term Capital Management’s Big Loss”)
- (Yacine Aït-Sahalia and Jean Jacod, *High-Frequency Financial Econometrics*, “Models for Market Microstructure Noise / Additive Noise” (Roll model))
- (Treasury Market Practices Group (TMPG), *U.S. Treasury Securities Fails Charge Trading Practice*, revised 2018-04-23)
- (Federal Reserve Bank of New York, *Primary Dealer Policy and Procedures*; *Primary Dealer List*, accessed 2026-02-15)
- (Board of Governors of the Federal Reserve System, *Monetary Policy Report*, 2020-06, “Box: U.S. Treasury Market Liquidity and Functioning”)
