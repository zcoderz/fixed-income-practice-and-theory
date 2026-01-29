# Chapter 9: Repo — The Bond Market's Funding Engine

---

## Introduction

Imagine you are a trading desk at a major dealer. A client walks in wanting to sell $100 million face of a five-year Treasury. You buy the bonds—but now you have a problem. Where does the cash come from? Drawing on your institution's capital is expensive and ties up scarce balance sheet. The solution, used every day across trillions of dollars of positions, is deceptively simple: you borrow the purchase amount from someone who has excess cash, post the bonds you just bought as collateral, and agree to buy them back tomorrow (or next week, or next month) at a slightly higher price. That price difference is the interest on your loan.

This is a **repurchase agreement**—universally called **repo**—and it is the heartbeat of the fixed income markets. Repo serves three critical functions simultaneously: it provides secured short-term funding for dealers and investors; it enables short selling by allowing market participants to borrow specific securities; and it links spot and forward prices through explicit arbitrage relationships. Understanding repo is not optional for anyone working in rates or credit. Every funded position you take, every short you execute, and every carry calculation you perform runs through the repo market.

The importance of repo extends beyond individual trades to systemic benchmarks. Hull notes that "the secured overnight financing rate (SOFR) is an important volume-weighted median average of the rates on overnight repo transactions in the United States." This means the reference rate that now underlies hundreds of trillions of dollars in derivatives and loans is itself a repo rate—making repo mechanics foundational to modern fixed income.

This chapter covers:

1. **Repo mechanics**: start leg, end leg, and the repurchase price formula
2. **Financing longs and shorting via reverse repo**: how desks use repo to manage inventory and execute shorts
3. **Carry**: the decomposition of funded bond P&L into price change and financing cost
4. **General collateral vs specials**: why some bonds finance cheaper than others, and what drives specialness
5. **Haircuts and margin**: how lenders protect themselves, and the leverage implications
6. **Repo as a forward-pricing link**: how spot, repo, and forward prices are connected by no-arbitrage
7. **SOFR and the modern reference rate**: why repo became the foundation for USD interest rates
8. **Market stress and the limits of specialness**: what happens when collateral becomes scarce

The conventions and formulas developed here will reappear throughout the book—in Treasury futures pricing (Chapter 23), in swap spread analysis (Chapter 27), and wherever carry enters a trade.

---

## 9.1 What Is a Repurchase Agreement?

### 9.1.1 The Economic Substance: A Secured Loan

Tuckman provides the essential framing: repo allows entities to effect a short-term loan backed by securities as collateral. The typical transaction arises when one party has cash to invest and another has securities to pledge. In an overnight repurchase agreement, a corporation with excess cash "would purchase $100 million worth of securities from the borrower and agree to sell them back the next day for a higher price."

The mechanics involve two legs:

**Start leg (initiation date):** Cash is exchanged for securities. The cash borrower delivers collateral and receives funds; the cash lender delivers funds and receives collateral.

**End leg (maturity date):** The transaction reverses. The cash borrower returns the loan amount plus interest (the **repurchase price**) and takes back the collateral.

Economically, this is identical to a secured loan. As Tuckman notes, "this chapter neglects the legal treatment of repurchase agreements in the event of insolvency and does not differentiate between a repurchase agreement and a secured loan."

> **Analogy: The Pawn Shop**
>
> Repo is fundamentally a **Secured Loan**, just like a transaction at a Pawn Shop.
> *   **You**: The Dealer (Borrower) with a Bond.
> *   **Counterparty**: The Pawn Shop (Cash Lender).
> *   **Transaction**: You hand over the Watch (Bond) and get Cash. You promise to come back tomorrow with slightly more cash (Repurchase Price) to get your Watch back.
> *   **Haircut**: The Pawn Shop only gives you 98% of the watch's value, just in case the price drops before you return.
>
> *Key Difference*: In Repo, the "Pawn Shop" can re-pledge (use) your watch while they have it!

> **Legal note:** The legal status of repo has not been definitively settled as a securities trade versus collateralized borrowing. Tuckman explains that the structure—sell today, repurchase tomorrow—exists partly to ensure that if the borrower defaults, the lender can immediately sell the collateral without being stayed by bankruptcy proceedings. Market participants carefully avoid the terms "borrowing" or "lending" in documentation for this reason.

### 9.1.2 Repo Rate and the Repurchase Price Formula

The **repo rate** $r$ is quoted as a simple interest rate under the ACT/360 day count convention (for USD markets). If $L_0$ is the cash exchanged at the start leg and $d$ is the number of days to the end leg, the repurchase cash $L_1$ satisfies:

$$\boxed{L_1 = L_0 \left(1 + r \frac{d}{360}\right)}$$

This formula is identical to money market loan conventions. Hull notes that "because it is a secured rate, a repo rate is theoretically very slightly below the corresponding fed funds rate."

**Example from Tuckman:** A corporation invests $100 million overnight at a repo rate of 5.45%. The repurchase price is:

$$\$100{,}000{,}000 \times \left(1 + \frac{0.0545}{360}\right) = \$100{,}015{,}139$$

The corporation has effectively made a one-day loan at 5.45%, earning $15,139 in interest.

For a **term repo** of seven days at the same rate:

$$\$100{,}000{,}000 \times \left(1 + \frac{7 \times 0.0545}{360}\right) = \$100{,}105{,}972$$

**Unit check:** Rate $r$ is per year, $d/360$ is in years, so $r \times d/360$ is dimensionless. Multiplying by dollars gives dollars.

**Sanity checks:**
- If $d = 0$, then $L_1 = L_0$ (no interest for zero days)
- If $r$ increases, $L_1$ increases linearly for fixed $d$ (higher rate = more interest)

### 9.1.3 Overnight, Term, and Open Repo

**Overnight repo** settles today and matures the next business day. This is the most liquid segment of the market and the building block for the **Secured Overnight Financing Rate (SOFR)**. Hull describes SOFR as "an important volume-weighted median average of the rates on overnight repo transactions in the United States." Since LIBOR's phase-out, SOFR has become the primary USD reference rate, making overnight repo rates foundational to all interest rate markets.

**Term repo** has a maturity beyond overnight—typically one week, one month, or up to a few months. Tuckman notes that the term market "allows borrowers and lenders of cash to lock in a fixed rate over longer time periods, though typically less than a few months."

> **Practitioner Note (not sourced from books/):** **Open Repo**
>
> An **open repo** has no fixed maturity date. The rate resets daily (typically to the overnight GC rate), and either party can terminate with one day's notice. Open repos are popular for:
> - **Flexibility**: A desk unsure of its holding period avoids committing to a fixed term
> - **Liquidity management**: Cash investors can recall funds quickly
> - **Rate uncertainty**: Rolling overnight may be cheaper than term if the curve is inverted
>
> The trade-off is **rollover risk**: the counterparty can terminate unexpectedly, forcing you to find alternative financing. In stress, open repos are the first to get pulled.

> **Desk Reality: Overnight vs Term vs Open**
>
> **When do desks use each?**
> - **Overnight**: Default choice for day-to-day inventory financing. Maximum liquidity, but must roll daily.
> - **Term**: Lock in funding for a known holding period (e.g., over quarter-end, over a Fed meeting). Pays for certainty.
> - **Open**: When uncertain about position duration. Common for relative value trades with unclear exit timing.
>
> The decision balances **rate cost** (term usually higher than overnight) against **rollover risk** (overnight rate can spike). Many desks got caught in 2019 when overnight repo spiked to 10% and they had funded term positions with O/N.

### 9.1.4 SOFR: The Modern Reference Rate

Hull explains that SOFR replaced LIBOR as the primary USD reference rate precisely because it is anchored in observable repo transactions rather than bank estimates. The rate is a volume-weighted median of overnight Treasury repo rates, making it both transparent and manipulation-resistant.

**SOFR Compounding Formula (Hull):**

For periods longer than overnight, SOFR rates are determined by compounding daily rates. Hull provides the formula: if the overnight SOFR rate on business day $i$ is $r_i$ and applies for $d_i$ calendar days, the annualized rate for a period of $D$ total calendar days is:

$$\boxed{R = \left[\prod_{i=1}^{n}\left(1 + r_i \frac{d_i}{360}\right) - 1\right] \times \frac{360}{D}}$$

where $n$ is the number of business days in the period.

**Example:** Suppose over a 5-day period (Friday to Wednesday), the overnight SOFR rates are:
- Friday: 5.00% (applies for 3 days: Sat, Sun, Mon)
- Monday: 5.05% (applies for 1 day)
- Tuesday: 5.10% (applies for 1 day)

The compounded rate is:
$$R = \left[(1 + 0.0500 \times 3/360)(1 + 0.0505 \times 1/360)(1 + 0.0510 \times 1/360) - 1\right] \times \frac{360}{5}$$
$$R = \left[(1.0004167)(1.0001403)(1.0001417) - 1\right] \times 72 = 0.05033 = 5.033\%$$

> **Desk Reality: Why SOFR Matters for Middle Office**
>
> If you work in risk, operations, or product control, you'll encounter SOFR in:
> - **Discounting**: OIS discounting curves are now SOFR-based
> - **Loan payments**: Corporate loans reference SOFR + spread
> - **Swap settlements**: SOFR swaps are the new standard
>
> Understanding that SOFR is a repo rate—specifically the rate at which the broadest set of Treasury repo transactions cleared—helps you understand why it's considered "risk-free" (it's secured by Treasuries) and why it can spike during funding stress (repo markets are the mechanism).

---

## 9.2 Financing Long Positions: Selling the Repo

### 9.2.1 The Dealer's Problem

When a trading desk buys bonds from a client, it needs cash to pay for them. Rather than using the institution's capital, the desk will **repo out** the securities (also called "sell the repo"). Tuckman describes the typical borrowers: "financial institutions in the business of making markets in U.S. government securities."

The terminology matters because it indicates the economic position:
- **Sell the repo** = deliver securities, receive cash, agree to repurchase later
- You are the **cash borrower** and **securities lender**

### 9.2.2 A Complete Example

Tuckman provides a detailed example that illustrates the full mechanics. Assume:
- Trade date: February 14, 2001; settlement: February 15, 2001
- Security: $100 million face of the 5⅞s of November 15, 2005
- Bid price: 103-18 (= 103.5625)
- Accrued interest: 1.493094 (92 days into a 181-day coupon period)

The invoice price due to the seller (a mutual fund) is:

$$\$100{,}000{,}000 \times (103.5625 + 1.493094)\% = \$105{,}055{,}594$$

The trading desk borrows this amount overnight at the market repo rate of 5.10%. On February 16, the desk owes:

$$\$105{,}055{,}594 \times \left(1 + \frac{0.051}{360}\right) = \$105{,}055{,}594 + \$14{,}883 = \$105{,}070{,}477$$

**The cost of financing the overnight position is $14,883.**

If no buyer emerges the next day, the desk must **roll** its repo—either extending the existing agreement or finding another counterparty willing to lend against the bonds. As Tuckman observes, the desk "might simply extend the term of the original agreement... or sell the repo to another repo player, like a municipality."

### 9.2.3 Mapping Cash to Invoice Price

The cash borrowed in repo is typically set equal to (or close to) the **invoice price** of the collateral:

$$L_0 \approx N \times I(0) = N \times (P(0) + AI(0))$$

where $N$ is the face amount, $P(0)$ is the clean price, and $AI(0)$ is the accrued interest. In practice, **haircuts** (discussed in Section 9.8) reduce this amount to provide overcollateralization.

---

## 9.3 Shorting Securities: Buying the Repo (Reverse Repo)

### 9.3.1 The Mechanics of a Short Sale

A trading desk that sells bonds it doesn't own—going **short**—must deliver those bonds to the buyer. The solution is a **reverse repurchase agreement**: the desk lends cash to someone who owns the bonds, takes the bonds as collateral, and delivers them to the buyer.

Tuckman explains: "The trading desk finds some party that owns the [bonds], perhaps another investment bank; lends that bank the cash received from the [buyer]; takes the [bonds] as collateral; and, finally, delivers that collateral to the [buyer]."

The terminology reflects the mirror-image nature of the transaction:
- **Repo (sell the repo):** You deliver securities, receive cash, and agree to repurchase. You are the cash borrower.
- **Reverse repo (buy the repo):** You deliver cash, receive securities, and agree to resell. You are the cash lender—but your motivation is to obtain specific securities.

As Tuckman notes, "there is no difference between a reverse repurchase agreement from the point of view of the trading desk and a repurchase agreement from the point of view of the other investment bank. Nevertheless, the term reverse repo is useful to emphasize that the lender of cash is motivated by the need to borrow particular bonds."

### 9.3.2 Covering a Short

The purchase and delivery of securities that had been sold and borrowed is called **covering a short**. If the desk cannot find a seller, it must **roll its short**—extending the repo or finding another counterparty willing to lend the bonds.

### 9.3.3 Net Borrowers of Cash

Tuckman observes that "on average across the money market, brokers and dealers are net borrowers of cash to finance their inventories." The repo market serves primarily as a funding mechanism for dealer balance sheets, though the reverse repo market is equally active for securities lending.

> **Visualization: The Shorting Engine**
>
> Short selling relies on a continuous loop of financing.
> 1.  **Reverse Repo**: You Borrow the Bond (and Lend Cash).
> 2.  **Market Sale**: You Sell the Bond to the Market (and Receive Cash).
> 3.  **Invest**: You invest the Cash (earn Rebate).
>
> *Result*: You pay the Repo Rate (to borrow the bond), and you earn the Rebate Rate (on your cash). The difference is your **Cost of Carry**. Repo is the engine room of *all* leveraged finance.

---

## 9.4 Carry: Decomposing Funded Bond P&L

### 9.4.1 The Definition of Carry

**Carry** is defined as interest income on a position minus the cost of financing that position. This is the core concept for understanding P&L on funded positions. Tuckman provides the definitive formulation: "Practitioners like to divide the profit or loss of a trade into a component due to price changes and a component due to carry."

Let:
- $P(0), P(d)$: Clean prices at initiation and $d$ days later
- $AI(0), AI(d)$: Accrued interest at initiation and $d$ days later
- $r$: Repo rate
- $c$: Coupon rate
- $D$: Actual days in the coupon period

The P&L from buying the bond, financing at repo, and selling after $d$ days is:

$$\boxed{\text{P\&L} = P(d) - P(0) + \underbrace{[AI(d) - AI(0)]}_{\text{Interest income}} - \underbrace{(P(0) + AI(0)) \times r \times \frac{d}{360}}_{\text{Financing cost}}}$$

This decomposes as:

$$\text{P\&L} = \text{Price change} + \text{Carry}$$

where:

$$\boxed{\text{Carry} = \text{Interest income} - \text{Financing cost}}$$

### 9.4.2 Numerical Example

From the Tuckman example of Section 9.2, suppose the desk sells the bond the next day at the ask price (one tick higher than bid). The desk earns $32,596 on the trade:

- **Price change (bid-ask spread):** $\$100{,}000{,}000 \times (1/32)\% = \$31{,}250$
- **Interest income (one day's accrual):** $\$100{,}000{,}000 \times (1.509323 - 1.493094)\% = \$16{,}229$
- **Financing cost:** $\$14{,}883$ (from Section 9.2)
- **Carry:** $\$16{,}229 - \$14{,}883 = \$1{,}346$
- **Total P&L:** $\$31{,}250 + \$1{,}346 = \$32{,}596$

### 9.4.3 When Is Carry Positive?

Tuckman explains: "the carry in this trade is positive because the coupon rate of the bond, 5.875%, is greater than the repo rate, 5.10%." However, the relationship is not exact:

1. Interest income is earned on **face value**, while financing cost is applied to the **full (invoice) price**
2. Interest income uses **actual/actual** day count, while repo uses **actual/360**

These differences can cause carry to be slightly different from a naive coupon-minus-repo calculation.

### 9.4.4 Carry and Shorts

For a short position, the signs flip: the desk **pays** the coupon rate (to the securities lender) and **earns** the repo rate. Tuckman's example shows that the same trade shorted earns only $29,908, compared to $32,596 for the long—the difference is the negative carry of approximately $1,342 (plus rounding).

### 9.4.5 Breakeven Calculations

Carry is useful for computing **breakeven price changes**. Tuckman shows that an investor holding the 5⅞s for 30 days at 5.10% repo earns:

- Interest income: $\$486{,}878$
- Financing cost: $\$446{,}688$
- Carry: $\$40{,}190$

The bond price can fall by about 4 cents per 100 face before the investment loses money.

Similarly, for shorts, carry determines **breakeven holding periods**. A short expecting a price fall from 105.055594 to 105 must see the move within about 41 days before negative carry offsets the price gain.

### 9.4.6 Carry Does Not Determine Expected Returns

Tuckman cautions: "Positive carry trades have the desirable property that they earn money as they go. But this by no means implies that the expected return of a positive carry trade is greater than that of a negative carry trade."

A premium bond has positive carry but faces price pull-to-par; a discount bond has negative carry but gains from price appreciation. The expected return of any fairly priced portfolio equals the short-term rate plus a risk premium, whether that return comes from carry or price change. See Chapter 7 for a fuller treatment of return decomposition.

### 9.4.7 P&L Attribution in Practice

> **Desk Reality: The Daily P&L Explain**
>
> Every trading desk receives a daily P&L report that breaks down where money was made or lost. For a funded bond position, the typical attribution shows:
>
> | Component | What It Measures |
> |-----------|------------------|
> | **Price P&L** | Change in clean price × position size |
> | **Accrual P&L** | Coupon income accrued that day |
> | **Financing P&L** | Repo cost (or income for shorts) |
> | **Specialness P&L** | Difference between GC and actual repo rate |
>
> **Why middle office cares:** When there's a P&L break, it's often because:
> - Repo rate used in explain doesn't match actual rate
> - Day count mismatch between systems
> - Specialness not captured (especially for shorts)
>
> The formula: $\text{Total P\&L} = \text{Price P\&L} + \text{Accrual P\&L} - \text{Financing P\&L} + \text{Specialness P\&L}$

---

## 9.5 General Collateral vs Specials

### 9.5.1 What Drives Repo Rates for Different Collateral?

Not all Treasury securities finance at the same rate. Tuckman distinguishes two types of collateral demand:

**General collateral (GC):** Investors who "do not usually care about which particular Treasury securities they take as collateral. These investors are said to accept general collateral." The **GC rate** is the prevailing rate for repos where the cash lender accepts any Treasury.

**Special collateral (specials):** Participants who "do care about the specific issues used as collateral." Trading desks financing specific bonds, or traders needing to borrow specific securities to cover shorts, require special collateral. The **special rate** for a particular bond can be significantly below GC.

### 9.5.2 The GC Rate and Fed Funds

Tuckman notes that on February 15, 2001, the GC rate was 5.44%, "6 basis points below the fed funds target rate of 5.50%. The GC rate is typically below the fed funds target rate because loans through repurchase agreement are effectively secured by collateral, while loans in the fed funds market are not."

Hull similarly explains: "Because it is a secured rate, a repo rate is theoretically very slightly below the corresponding fed funds rate." This spread reflects the credit protection provided by collateral.

### 9.5.3 The Specialness Spread

Define the **specialness spread**:

$$\boxed{s = r_{\text{GC}} - r_{\text{spec}}}$$

A bond with a large specialness spread is "very special"—it finances at a rate well below GC.

> **Analogy: Why is Specialness Weird? (The Antique Watch)**
>
> Normally, if you borrow money, you pay interest.
> But what if you have a **Rare Antique Watch** (a Special Bond) that the Pawn Shop owner (Short Seller) desperately *needs* to complete his collection (Cover a Short)?
>
> *   **Result**: He might lend you cash at **0% interest**, or even **pay you** to hold your watch!
> *   **Insight**: Specialness is the "Rental Fee" for the Bond disguised as a "Cheap Loan" for the Cash.

**Example from Tuckman (February 15, 2001):**

| Treasury Issue | Comment | Special Rate | Specialness |
|---------------|---------|--------------|-------------|
| 5.75% Nov 15, 2005 | On-the-run 5-year | 3.85% | 159 bp |
| 5.75% Aug 15, 2010 | On-the-run 10-year | 4.25% | 119 bp |
| 4.75% Jan 31, 2003 | On-the-run 2-year | 4.88% | 56 bp |

The OTR 5-year financed at 159 basis points below GC—someone taking it as collateral was willing to earn only 3.85% instead of 5.44% in order to obtain that specific bond.

### 9.5.4 Why Do Specials Arise?

Tuckman identifies several sources:

1. **Liquidity and shorting demand:** "Current issues tend to be more liquid... Most shorts in Treasuries are for relatively brief holding periods." The on-the-run issues are ideal for shorts because they can be covered quickly at low transaction cost.

2. **The liquidity premium creates shorts:** "Investors and traders long an on-the-run security for liquidity reasons require compensation to sacrifice liquidity by lending those securities in the repo market. At the same time, investors and traders wanting to short the on-the-run securities are willing to pay for the liquidity of shorting these securities when borrowing them."

3. **Arbitrage activity:** Arbitrage traders who view a sector as rich relative to swaps may build a "large short base" in that sector, causing issues to trade special.

4. **Supply shocks:** "A large sale of a particular security from the dealer community to an investor that does not participate in the repo market might suddenly make it difficult for shorts to borrow that security."

### 9.5.5 The Dollar Value of Specialness

For someone who owns a special bond, the financing advantage is real money. If you can borrow at 159 bp below GC:

$$\text{Daily benefit} = L_0 \times \frac{s}{360} = \$100{,}000{,}000 \times \frac{0.0159}{360} = \$4{,}417 \text{ per day}$$

Conversely, if you're short a special bond, you must lend cash at below-market rates to obtain the bond—specialness is a cost.

**Worked Example: The Cost of Shorting a Special**

Suppose you're short $50 million face of a bond trading 150 bp special (GC = 5.00%, special rate = 3.50%). Your daily cost of maintaining the short:

$$\text{Daily specialness cost} = \$50{,}000{,}000 \times \frac{0.0150}{360} = \$2{,}083 \text{ per day}$$

Over 30 days, this costs $62,500—money paid just to borrow the bond, before any price movement. This is why traders say "don't fight specialness."

### 9.5.6 Repo as a "Bond Thermostat": Reading the Market

> **Desk Reality: The Bond Thermostat**
>
> Specialness spreads function as a real-time **thermometer** for short interest in a bond. When specialness spikes:
>
> | Specialness Level | What It Signals |
> |-------------------|-----------------|
> | 0-25 bp | Normal; modest short interest |
> | 25-75 bp | Elevated; trade is crowded |
> | 75-150 bp | Hot; significant shorting activity |
> | 150+ bp | Danger zone; squeeze risk |
>
> **Trading rule:** When a bond goes 100+ bp special, the short is "crowded"—many traders have the same position. This creates **squeeze risk**: if shorts start covering, price can spike violently as everyone rushes for the exit simultaneously.
>
> **For middle office:** When you see a bond's specialness spike in your financing report, alert the trading desk. It may signal an impending squeeze or a need to reduce position.

### 9.5.7 Anatomy of a Squeeze

A **squeeze** occurs when shorts cannot borrow enough of a specific bond to cover their positions, forcing a disorderly rush to buy in the market. The mechanics unfold systematically:

1. **Market builds a short base:** Traders sell a bond short, expecting prices to fall. Each short requires borrowing the bond via reverse repo.

2. **Borrowing supply dries up:** Natural lenders (institutions holding the bond) reduce lending—perhaps due to credit concerns, operational disruption, or simply deciding to hold.

3. **Specialness spikes:** As borrowing becomes difficult, those needing the bond accept lower and lower repo rates. Specialness rises toward its limit.

4. **Fails increase:** When specialness hits the floor (see Section 9.6.4), shorts cannot borrow at any price. They fail to deliver bonds they've sold.

5. **Shorts forced to cover:** To avoid mounting fails and the associated penalties, shorts buy bonds in the cash market. This demand pushes prices up.

6. **Price spike:** As multiple shorts cover simultaneously, prices can gap higher dramatically. The squeeze "breaks" when enough bonds are delivered to satisfy demand.

**The September 2001 squeeze** (Section 9.10) illustrates this pattern: operational disruption reduced lending supply, specialness spiked to near-zero rates, fails surged, and prices of on-the-run issues became severely dislocated.

---

## 9.6 Special Repo Rates and the Auction Cycle

### 9.6.1 The Pattern of On-the-Run Specialness

Tuckman presents data showing that special spreads for on-the-run securities "tend to be small immediately after auctions and to peak before auctions." The pattern reflects supply and demand dynamics:

**Immediately after an auction:** The new on-the-run issue has just been distributed. Shorts can stay in the previous on-the-run or shift to the new one—this substitutability depresses special spreads. After a **reopening** (an auction that increases the size of an existing issue), extra supply also depresses spreads.

**As time passes:** "Shorts tend to migrate toward the on-the-run security and its special spread tends to rise."

### 9.6.2 Volatility and Magnitude

From Tuckman's data: "The special spreads are quite volatile on a daily basis, reflecting supply and demand for special collateral on that day. Second, special spreads can be quite large: Spreads of 200 to 400 basis points are quite common."

### 9.6.3 The Floor on Special Rates: The Pre-2009 Logic

Tuckman explains the traditional argument for why special rates cannot go meaningfully negative:

"Consider a trader who is short the OTR 10-year and needs to borrow it through a repurchase agreement. If for some reason the bond cannot be borrowed, the trader will fail to deliver it and, consequently, not receive the proceeds from the sale. In effect, the trader will lose one day of interest on the proceeds."

If the special rate is 0%, borrowing the bond earns the same as failing—nothing. If the special rate went **negative**, the trader would have to pay to lend cash, which is worse than failing. Therefore, under this logic:

$$r_{\text{spec}} \geq 0\% \quad \Rightarrow \quad s \leq r_{\text{GC}}$$

In the fall of 2001, with GC near 2%, the maximum special spread was about 200 basis points.

### 9.6.4 Fails Charges and “Borrow vs Fail” Economics (Modern Intuition)

When a short cannot borrow a security, it may **fail** to deliver. Historically (the pre-2009 logic), failing was often described as being “equivalent” to lending cash at 0%: you don’t deliver, so you don’t receive proceeds, and you miss out on investing those proceeds for a day. That logic suggests specials shouldn’t go meaningfully below 0%.

In modern Treasuries settlement practice, an explicit **fails charge** can apply to discourage chronic failing (see Chapter 1 and the TMPG trading practice). This changes the economics:

- Failing now has an **explicit penalty component** (not just an opportunity cost).
- As a result, specials can go **very low** and may even go **negative** in extreme scarcity/squeeze episodes.

**Key desk takeaway:** There is no single, universal “special repo floor.” Instead, there is a *borrow-versus-fail tradeoff*:

- **Borrow** if the all-in cost of borrowing the bond (how much interest you forgo/“pay” by lending cash at a low special rate) is **less** than the all-in cost of failing (opportunity cost plus any fails charge).
- **Fail** if borrowing becomes even more expensive than failing.

That breakeven depends on the reference rate used in the fails charge, any floors in the rule, and the exact settlement convention—so treat any hard-coded number like “-3%” as, at best, an order-of-magnitude intuition, not a theorem.

---

## 9.7 Valuing a Bond Trading Special in Repo

### 9.7.1 The Relative Value Question

Tuckman presents an important application: how should an investor choose between two similar bonds when one trades special? Consider a money manager deciding between the on-the-run 5-year (yielding 4.970%) and an off-the-run issue with the same maturity (yielding 5.020%). The OTR is five basis points rich—but it also finances at 159 basis points below GC.

### 9.7.2 Components of the Relative Value

Tuckman's framework breaks the decision into three components:

1. **Liquidity value**: How much is the OTR's superior liquidity worth to this investor? A trader who expects to sell quickly values liquidity highly; a buy-and-hold investor values it less.

2. **Financing advantage**: If the OTR is expected to trade 100 bp special on average over a 90-day horizon, the financing benefit is approximately:

$$\frac{0.01 \times 90}{360} = 0.25\%$$

of face value.

3. **Yield rolldown**: If the OTR's yield premium is expected to narrow from 5 bp to 3 bp over the horizon, this creates a price loss as the OTR converges toward fair value.

### 9.7.3 The Trade-Off Calculation

Tuckman's example shows how to net these effects. If the financing advantage is 0.25% of face, but the anticipated yield convergence costs about 0.085% (based on DV01), and the coupon disadvantage costs about 0.031%, then the net advantage of the OTR is approximately:

$$0.25\% - 0.031\% - 0.085\% = 0.134\%$$

This is equivalent to about 3.1 basis points. But if the OTR trades 4 basis points rich (after accounting for 1 bp of liquidity value), the off-the-run is the better investment for this particular investor.

**Key insight:** Any trade or hedge involving a security that is trading special requires the same set of assumptions and calculations. How much is liquidity worth? How will special spreads behave? How will the premium change over time?

---

## 9.8 Haircuts, Repricing, and Leverage

### 9.8.1 Why Haircuts Exist

Even with Treasury collateral, a lender faces risk: the borrower might default at the same time Treasury prices decline. Selling the collateral might not cover the loan. Tuckman explains:

"Repo agreements include haircuts requiring the borrower of cash to deliver securities worth more than the amount of the loan. Furthermore, repo agreements often include repricing provisions requiring the borrower of cash to supply extra collateral in declining markets and allowing the borrower of cash to withdraw collateral in advancing markets."

### 9.8.2 Haircut Mechanics

Under one common convention, the haircut $h$ is a percentage reduction of collateral market value:

$$\boxed{L_0 = (1 - h) \times N \times I(0)}$$

where $N \times I(0)$ is the market value of the collateral.

**Alternative conventions warning:** Some markets define haircut as (Collateral Value − Loan)/Loan rather than (Collateral Value − Loan)/Collateral Value. Always verify the convention in use.

### 9.8.3 Equity and Leverage

The equity (unfunded portion) posted by the repo borrower is:

$$E_0 = N \times I(0) - L_0 = h \times N \times I(0)$$

This implies leverage of:

$$\boxed{\text{Leverage} = \frac{N \times I(0)}{E_0} = \frac{1}{h}}$$

**Example:** A 2% haircut implies 50× leverage. A 5% haircut implies 20× leverage.

### 9.8.4 Haircuts by Collateral Type

> **Practitioner Note (not sourced from books/):** Typical Haircuts
>
> Haircuts vary dramatically by collateral quality. Approximate ranges:
>
> | Collateral Type | Typical Haircut | Implied Leverage |
> |-----------------|-----------------|------------------|
> | Treasury bills | 0.5-1% | 100-200× |
> | Treasury notes/bonds | 1-2% | 50-100× |
> | Agency MBS | 2-5% | 20-50× |
> | Investment-grade corporates | 5-10% | 10-20× |
> | High-yield corporates | 10-25% | 4-10× |
> | Equities | 20-50% | 2-5× |
>
> **Stress adjustment:** During market stress, haircuts increase—sometimes dramatically. In 2008, some repo haircuts doubled or tripled overnight, forcing immediate deleveraging.

### 9.8.5 Margin Calls and Repricing

If collateral value falls, the loan may exceed the permitted amount under the haircut. The borrower must either post additional collateral or repay cash—this is a **margin call**.

**Example:** Collateral market value falls from $103 million to $101 million with a 2% haircut. Maximum permitted loan is $0.98 \times 101 = \$98.98$ million. If the outstanding loan is $100.94 million, the margin call is $1.96 million.

This creates **liquidity risk**: price declines generate immediate cash demands. In stress scenarios, inability to meet margin calls can force liquidation at unfavorable prices.

> **Desk Reality: The Margin Call Cascade**
>
> In market stress, margin calls create a dangerous feedback loop:
>
> 1. Prices fall → collateral values decline
> 2. Margin calls go out → borrowers must post cash or sell
> 3. Forced selling → prices fall further
> 4. Haircuts increase → even more margin calls
> 5. **Result**: A deleveraging spiral
>
> This is why 2008 was so severe: falling prices → rising haircuts → forced selling → falling prices. The repo market seized because lenders refused to roll repos at *any* haircut.
>
> **For middle office:** Monitor daily margin calls carefully. A spike in margin calls can be an early warning of stress.

**Worked Example: Forced Deleveraging from Haircut Increase**

A fund has $100 million in bonds, financed with 2% haircut ($98mm loan, $2mm equity).

Haircut increases to 5%. Maximum loan is now $95mm. The fund must:
- Repay $3mm of the loan, OR
- Sell $3mm / 0.95 = $3.16mm of bonds to stay within the new haircut

If the fund lacks $3mm cash, it must sell—potentially at distressed prices.

---

## 9.9 Repo as a Forward-Pricing Link

### 9.9.1 The Replication Argument

A fundamental insight is that buying a bond spot and financing it through repo replicates a **forward position** in the bond. Tuckman demonstrates: "a long forward position in the bond can be replicated by buying the bond spot and selling the repo. Once again, borrowing the purchase price to the forward date essentially locks in the cost of the bond as of the forward date."

### 9.9.2 Forward Invoice Price (No Intermediate Coupon)

Under no-arbitrage, the forward invoice price equals the terminal value of the repo loan:

$$\boxed{P_{\text{fwd}} + AI(d) = (P(0) + AI(0)) \times \left(1 + r \frac{d}{360}\right)}$$

Rearranging:

$$P_{\text{fwd}} = (P(0) + AI(0)) \times \left(1 + r \frac{d}{360}\right) - AI(d)$$

Or equivalently:

$$\boxed{P_{\text{fwd}} = P(0) - \text{Carry}}$$

This is Tuckman's equation (16.8): "when carry is positive the forward price is less than the spot price."

### 9.9.3 Forward Price with an Intermediate Coupon

If a coupon is paid during the forward period, the analysis becomes more complex. The bond owner continues to receive the coupon; the repo borrower must pass it through (a "manufactured coupon"). Economically, the repo loan balance is reduced by the coupon amount.

Let:
- $d_1$ = days from initiation to coupon date
- $d_2$ = days from coupon date to delivery
- $C$ = coupon payment (e.g., $c/2$ for a semiannual bond)

Then:

$$\boxed{P_{\text{fwd}} + AI(d) = \left[(P(0) + AI(0))\left(1 + r\frac{d_1}{360}\right) - C\right]\left(1 + r\frac{d_2}{360}\right)}$$

Tuckman explains the economic logic: "a bond owner who lends a bond through a repurchase agreement continues to receive that bond's coupon payments. The trader who borrowed and sold the bond in this example must make the... coupon payment to the lender of the bond."

**Sanity check:** If $C = 0$, this collapses to the no-coupon formula.

### 9.9.4 Implied Repo Rate

The relationship can be inverted. Given observed spot and forward prices, the **implied repo rate** is the financing rate that makes the arbitrage hold:

$$\boxed{r_{\text{implied}} = \frac{P_{\text{fwd}} + AI(d) - (P(0) + AI(0))}{(P(0) + AI(0))} \times \frac{360}{d}}$$

**Interpretation:** The implied repo rate is the financing cost "baked into" the forward price. If implied repo exceeds actual repo, the forward is cheap—you can buy spot, finance at actual repo, and sell forward for a profit.

**Worked Example: Computing Implied Repo**

Spot invoice price: $102.50
Forward price (30 days): $102.35 (plus $0.40 accrued at delivery = $102.75 invoice)
Days to delivery: 30

$$r_{\text{implied}} = \frac{102.75 - 102.50}{102.50} \times \frac{360}{30} = 0.00244 \times 12 = 2.93\%$$

If actual repo is 3.00%, the forward is 7 bp "rich" (overpriced relative to cash-and-carry arbitrage).

### 9.9.5 Preview: The Basis Trade

The implied repo concept is central to **Treasury futures basis trading**, covered fully in Chapter 23. A brief preview:

Tuckman (Ch 20) defines:
- **Gross basis** = Spot price − (Futures price × Conversion Factor)
- **Net basis** = Gross basis − Carry

Mathematically:
$$GB^i(t) = P^i(t) - cf^i \times F(t)$$
$$NB^i(t) = P^i_{fwd}(t) - cf^i \times F(t)$$

A **basis trade** involves:
1. Buy the bond spot
2. Finance via repo to the delivery date
3. Sell futures

If the implied repo rate in the futures exceeds your actual financing rate, this trade earns the spread. Tuckman notes: "Buying or selling the basis in this form involves no cash outlay: The repo position finances or invests the bond proceeds."

> **Desk Reality: The Hedge Fund Basis Trade**
>
> The basis trade is one of the largest fixed income arbitrage strategies. Hedge funds with cheap financing (low repo rates) buy Treasury bonds and sell futures, earning the implied-vs-actual repo spread.
>
> **The risk**: This trade is highly levered (50-100× on Treasuries). If repo rates spike or the basis widens adversely, the fund faces margin calls on both the repo and the futures. In March 2020, forced unwinds of basis trades contributed to Treasury market dysfunction.
>
> Chapter 23 develops the full mechanics, including conversion factors, cheapest-to-deliver, and delivery option value.

---

## 9.10 Market Stress: September 11, 2001

### 9.10.1 Disruption in the Specials Market

Tuckman documents a remarkable episode following the September 11 terrorist attacks:

"The terrorist attack on the World Trade Center disrupted the specials market in two ways. First, the resulting confusion and the destruction of records caused many government bond transactions to fail, meaning that bonds sold were not delivered. Second, heightened uncertainty and credit concerns caused many participants in the repo markets to pull their securities from the repo market."

The combination created a severe shortage of on-the-run collateral.

### 9.10.2 Extreme Special Rates

On September 20, 2001, repo rates reached extraordinary levels:

| Treasury Issue | Comment | Special Rate |
|---------------|---------|--------------|
| GC rate | | 1.75% |
| 4.625% May 15, 2006 | OTR 5-year | 0.10% |
| 5.000% Aug 15, 2011 | OTR 10-year | 0.35% |
| 3.625% Aug 31, 2003 | OTR 2-year | 0.65% |

Tuckman observes: "the GC rate was 1.75% while the fed funds rate at the time was 3%. The widespread shortage of Treasury collateral widened the spread between fed funds and GC to 125 basis points."

The OTR 5-year was 165 basis points special—nearly the entire GC rate.

### 9.10.3 The Treasury's Response

On October 4, 2001, the Treasury took unprecedented action:

"The Treasury made a surprise announcement on the morning of October 4, 2001. It would auction $6 billion of the OTR 10-year at 1:00 p.m. that day. This was unprecedented in two ways. First, the Treasury usually keeps to a strict issuance calendar. Second, the Treasury almost always gives much more notice of auctions."

The response was immediate: 10-year futures prices fell sharply, the spread between OTR and old 10-year notes compressed from 5.2 to 3.7 basis points, and the OTR 10-year special spread fell about 100 basis points.

### 9.10.4 Lesson: Operational Frictions Matter

The September 2001 episode illustrates that repo markets can become severely stressed when collateral supply is disrupted. Fails, credit concerns, and collateral hoarding can drive special rates to their floor and dislocate relative value relationships. Understanding these dynamics is essential for risk management.

---

## 9.11 Practical Notes

### 9.11.1 Market Structure

> **Practitioner Note (not sourced from books/):** Bilateral vs Tri-Party Repo
>
> The sources focus on repo economics rather than operational structure. Here's a brief overview of how the market actually operates:
>
> | Structure | Description | Use Case |
> |-----------|-------------|----------|
> | **Bilateral repo** | Direct agreement between two parties; each manages collateral and settlement | Dealer-to-dealer; specific collateral trades |
> | **Tri-party repo** | Third party (BNY Mellon, JPM) acts as agent; handles collateral selection, valuation, and settlement | Dealer-to-cash-investor; large GC trades |
> | **GCF repo** | General Collateral Finance; netting through FICC | Interdealer; anonymous GC trading |
> | **Sponsored repo** | Buy-side firms access FICC clearing via dealer sponsorship (post-2017) | Hedge funds seeking netting benefits |
>
> **Why tri-party dominates GC:** For a money market fund lending $500mm overnight, tri-party is efficient—the agent selects eligible collateral, handles substitutions, and automates settlement. Bilateral would require managing specific CUSIPs manually.
>
> **Why bilateral for specials:** When you need a specific bond (to cover a short), you must negotiate bilaterally with someone who owns that bond.

### 9.11.2 Common Quoting and Contract Gotchas

| Issue | Notes |
|-------|-------|
| **Rate convention** | Repo uses simple interest with ACT/360 (USD). Bond accrual may use different day counts. |
| **Invoice vs clean** | Repo cash is typically based on invoice price, not clean price. |
| **Coupon during repo** | Coupon belongs to security lender; borrower must pass through a "manufactured coupon." |
| **Settlement timing** | Getting dates wrong creates P&L noise. Use consistent calendar engines. |

### 9.11.3 Implementation Pitfalls

| Pitfall | Mitigation |
|---------|------------|
| **Sign conventions** | Repo = receive cash today, pay later (financing cost). Reverse = pay cash today, receive later (interest income). |
| **Margining frequency** | Haircuts and repricing create path-dependent liquidity needs. |
| **Price source for margin** | The mark source and timing (intraday vs EOD) matters in practice. |

### 9.11.4 Sanity Checks

| Check | What to Verify |
|-------|----------------|
| **Cashflow reconciliation** | Initial cash + interest − coupon adjustments = repurchase cash |
| **Special rate reasonableness** | Specials can be very low (even negative). If you see extreme specials, sanity-check against “borrow vs fail” economics (fails charge rules) and your sign conventions |
| **Leverage check** | With haircut $h$, leverage should be near $1/h$ |
| **Sensitivity sign** | Higher repo rate → higher financing cost → $\partial \text{P\&L}/\partial r < 0$ for funded longs |

---

## Summary

1. **Repo is secured short-term funding**: Cash versus collateral today, reversed at a higher repurchase price embedding repo interest.

2. **Repo vs reverse repo** are the same transaction from opposite perspectives. Sign conventions matter for P&L.

3. **Dealers use repo** to finance long inventory and execute short sales without tying up balance sheet capital.

4. **GC repo** reflects general secured funding; **specials** reflect demand for particular securities, often trading at lower rates.

5. **Specialness** ($r_{\text{GC}} - r_{\text{spec}}$) measures scarcity value and has real dollar impact via financing costs.

6. **Fails charges matter for specials:** settlement fails economics affect the “borrow vs fail” decision and can limit how extreme specialness can get.

7. **Haircuts** create overcollateralization and limit leverage; repricing creates margin call liquidity risk.

8. **Carry** = interest income − financing cost. Positive carry when coupon rate exceeds repo rate (approximately).

9. **Repo links spot and forward prices**: buying spot and financing at repo replicates a forward position.

10. **Implied repo** reveals the financing rate embedded in forward/futures prices—key to basis trading.

11. **SOFR** is based on overnight repo rates, making repo foundational to the entire USD interest rate market.

12. **In stress**, collateral shortages can drive specials to extreme levels and dislocate relative value.

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| Repo | Sale and repurchase agreement; economically a secured loan | Core funding mechanism for fixed income |
| Reverse repo | Same as repo from cash lender's view | Used to borrow specific securities |
| GC rate | Rate for repos accepting any eligible collateral | Benchmark secured funding rate; feeds into SOFR |
| Special rate | Rate for repos requiring specific collateral | Reflects scarcity/short demand |
| Specialness spread | $r_{\text{GC}} - r_{\text{spec}}$ | Dollar value of owning special bonds |
| Haircut | Reduction in collateral value for lending | Controls leverage, protects lender |
| Carry | Interest income minus financing cost | Key component of funded P&L |
| Implied repo | Financing rate implied by spot/forward prices | Reveals embedded financing in positions |
| SOFR | Volume-weighted median of overnight repo rates | Primary USD reference rate |
| Fails charge | Explicit penalty for Treasury settlement fails (per TMPG trading practice) | Affects “borrow vs fail” economics and can limit extreme specialness |

---

## Notation for This Chapter

| Symbol | Definition |
|--------|------------|
| $L_0$ | Cash exchanged at repo start (principal) |
| $L_1$ | Repurchase cash at repo end |
| $r$ | Repo rate (simple annual, ACT/360) |
| $d$ | Days in repo term |
| $h$ | Haircut (fraction of collateral not financed) |
| $r_{\text{GC}}$ | General collateral repo rate |
| $r_{\text{spec}}$ | Special repo rate for a particular security |
| $s$ | Specialness spread ($r_{\text{GC}} - r_{\text{spec}}$) |
| $P$, $I$, $AI$ | Clean price, invoice price, accrued interest |
| $N$ | Face value (notional) |
| $r_{\text{implied}}$ | Implied repo rate from spot/forward relationship |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is a repo economically? | A secured loan: borrower posts securities as collateral and repays cash plus interest |
| 2 | What is a reverse repo? | The cash lender's side of a repo; used to borrow specific securities |
| 3 | What are the two legs of a repo? | Start leg (cash vs collateral) and end leg (repurchase at higher price) |
| 4 | Why is the repurchase price higher? | It embeds repo interest |
| 5 | Write the repurchase formula | $L_1 = L_0(1 + rd/360)$ under ACT/360 simple interest |
| 6 | What is the GC rate? | The repo rate for general (non-specific) Treasury collateral |
| 7 | What is a special repo? | A repo for a specific security, often at below-GC rates |
| 8 | Define specialness spread | $s = r_{\text{GC}} - r_{\text{spec}}$ |
| 9 | Why do on-the-run issues trade special? | High demand to short them for liquidity reasons |
| 10 | How do you calculate the dollar value of specialness? | $L_0 \times s \times d/360$ |
| 11 | What is a haircut? | A reduction in collateral value that creates overcollateralization |
| 12 | How does haircut relate to leverage? | Leverage ≈ 1/h |
| 13 | What is carry (funded position)? | Interest income minus financing cost |
| 14 | When is carry typically positive? | When coupon rate exceeds repo rate |
| 15 | Write the funding-adjusted P&L formula | $\text{P\&L} = N(P(d)+AI(d)) - N(P(0)+AI(0))(1+rd/360)$ |
| 16 | How does repo link to forward prices? | Forward invoice = spot invoice × (1 + rd/360), so $P_{\text{fwd}} = P(0) - \text{Carry}$ |
| 17 | What can limit how negative a special can trade? | If borrowing becomes more expensive than failing delivery (after accounting for fails charges and opportunity cost), some participants may choose to fail instead—limiting extreme specialness |
| 18 | What is a manufactured coupon? | The coupon payment passed from security borrower to lender during a repo |
| 19 | What is repo-rate risk? | P&L sensitivity to changes in financing rates |
| 20 | What is SOFR and why does it matter? | Secured Overnight Financing Rate—the primary USD reference rate based on repo |
| 21 | What is implied repo? | The financing rate embedded in the forward-spot price relationship |
| 22 | What does high specialness signal? | Crowded short interest—potential squeeze risk |

---

## Mini Problem Set

### Problems 1–7 (with Solution Sketches)

**Problem 1:** A repo starts with $L_0 = \$25{,}000{,}000$, $r = 3.75\%$, $d = 3$ days (ACT/360). Compute $L_1$ and interest.

*Solution:* $L_1 = 25{,}000{,}000 \times (1 + 0.0375 \times 3/360) = \$25{,}007{,}812.50$. Interest = $\$7{,}812.50$.

**Problem 2:** A bond has clean price 99.75 and accrued 1.10 per $100. For $N = \$50$ million face, compute invoice cash exchanged.

*Solution:* Invoice per $100 = 99.75 + 1.10 = 100.85$. Cash = $50{,}000{,}000 \times 1.0085 = \$50{,}425{,}000$.

**Problem 3:** When is carry positive for a funded long?

*Solution:* When coupon accrual exceeds financing cost—generally when coupon rate > repo rate, adjusting for invoice vs face and day counts.

**Problem 4:** For $N = \$100$ million, invoice 102%, $d = 7$, compute P&L change for +10 bp repo rate move.

*Solution:* $\Delta \text{P\&L} \approx -100{,}000{,}000 \times 1.02 \times (7/360) \times 0.0010 = -\$1{,}983.33$.

**Problem 5:** Collateral market value is $80 million; haircut is 5%. Compute cash lent and leverage.

*Solution:* $L_0 = 0.95 \times 80{,}000{,}000 = \$76{,}000{,}000$. Equity = $\$4{,}000{,}000$. Leverage = $1/0.05 = 20\times$.

**Problem 6:** Using Problem 5, if collateral falls to $76 million, compute margin call.

*Solution:* Required max loan = $0.95 \times 76{,}000{,}000 = \$72{,}200{,}000$. Current loan from Problem 5 is $76{,}000{,}000$. Margin call (cash to post / loan to reduce) = $76{,}000{,}000 - 72{,}200{,}000 = \$3{,}800{,}000$.

**Problem 7:** $r_{\text{GC}} = 4.5\%$, $r_{\text{spec}} = 0.5\%$, $L_0 = \$200$ million, $d = 1$. Compute daily specialness benefit.

*Solution:* Benefit = $200{,}000{,}000 \times (0.045 - 0.005)/360 = \$22{,}222.22$.

### Problems 8–12 (No Solutions)

**Problem 8:** Build a 40-day repo spanning a coupon. Show cashflows under a stated coupon/loan adjustment convention.

**Problem 9:** Given spot invoice $I_0$ and repo rate $r$, derive the forward clean price formula.

*Hint:* Start from $P_{\text{fwd}} + AI(d) = I_0 \times (1 + rd/360)$. Rearrange to isolate $P_{\text{fwd}}$ and express in terms of spot clean price and carry.

**Problem 10:** Repeat Problem 9 with one coupon inside the forward horizon.

**Problem 11:** Describe two distinct risks when funding a term trade by rolling overnight repo.

**Problem 12:** A bond becomes more special after you short it. How does that affect your P&L and funding?

---

## References

- Bruce Tuckman, *Fixed Income Securities* (repo mechanics; specialness; funded P&L/carry; on-the-run vs off-the-run; basis intuition).
- John C. Hull, *Options, Futures, and Other Derivatives* (SOFR as a repo-based overnight reference; money-market conventions).
- Treasury Market Practices Group (TMPG), *U.S. Treasury Securities Fails Charge Trading Practice* (settlement fails charges and their interaction with “borrow vs fail” economics).

---

*Cross-references:*
- Bond pricing and invoice/clean/dirty: Chapter 5
- Day count conventions: Chapter 4
- Carry and rolldown return decomposition: Chapter 7
- Treasury microstructure and on/off-the-run: Chapter 10
- Treasury futures and implied repo: Chapter 23
- Swap spreads and asset swaps: Chapter 27
