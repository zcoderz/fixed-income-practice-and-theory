# Chapter 9: Repo — The Bond Market's Funding Engine

---

## Introduction

Prerequisites: [Chapter 4 — Money-Market Building Blocks](chapter_04_money_market_building_blocks.md), [Chapter 5 — Fixed-Rate Bond Pricing](chapter_05_fixed_rate_bond_pricing.md), [Chapter 7 — Bond Return Decomposition](chapter_07_bond_return_decomposition.md)  
Follow-on: [Chapter 10 — Treasury Market Microstructure and Relative Value](chapter_10_treasury_microstructure_relative_value.md), [Chapter 11 — DV01/PV01](chapter_11_dv01_pv01_definitions_computation.md), [Chapter 23 — Treasury Futures](chapter_23_treasury_futures.md), [Chapter 27 — Swap Spreads and Asset Swaps](chapter_27_swap_spreads_asset_swaps_swap_curve_rv.md), [Chapter 28 — Basis Trades in Rates](chapter_28_basis_trades.md), [Chapter 33 — Collateral Discounting and OIS](chapter_33_collateral_discounting_ois.md)

Repo is the workhorse market for short-term, collateralized funding and for borrowing specific securities. A repo is a sale of securities for cash with an agreement to repurchase them later at a higher price. The repo agreement is a short-term collateralized loan.

Repos and reverse repos play an important role in the pricing of derivative securities because they allow short positions to be taken in bonds.

> With repo transactions, we can "buy" without really buying, and we can "sell" without really selling.

Because it is a secured rate, a repo rate is theoretically very slightly below the corresponding fed funds rate. The secured overnight financing rate (SOFR) is an important volume-weighted median average of the rates on overnight repo transactions in the United States.

## Learning Objectives
- After this chapter, you can describe repo and reverse repo as two views of the same secured financing trade.
- You can translate a repo quote into start/end cashflows and compute the repurchase price on a simple 360-day basis.
- You can compute funded carry for a bond position and interpret the sign and units of each term.
- You can explain GC vs special repo and interpret specialness as scarcity value (a convenience yield).
- You can compute and interpret a funding-rate risk scalar (`Repo01`) with explicit bump object, units, and sign.

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

Repo is best understood as a short-term loan backed by securities as collateral. One party provides cash and receives securities; the other party provides securities and receives cash. The agreement is unwound later at a **repurchase price** that is slightly higher than the start-leg cash—this difference is the repo interest.

The mechanics involve two legs:

**Start leg (initiation date):** Cash is exchanged for securities. The cash borrower delivers collateral and receives funds; the cash lender delivers funds and receives collateral.

**End leg (maturity date):** The transaction reverses. The cash borrower returns the loan amount plus interest (the repurchase cash) and takes back the collateral.

In this chapter we treat repo as economically equivalent to a secured loan and ignore legal/bankruptcy distinctions.

> **Analogy (pawn shop):** You pledge a bond for cash and later repay cash plus interest to get the bond back. A **haircut** means you receive less than the bond's market value.

### 9.1.2 Repo Rate and the Repurchase Price Formula

In this chapter we model repo interest as **simple interest on a 360-day basis**. If $L_0$ is the cash exchanged at the start leg and $d$ is the number of days to the end leg, the repurchase cash $L_1$ satisfies:

$$\boxed{L_1 = L_0 \left(1 + r \frac{d}{360}\right)}$$

This is the same simple-interest convention used for many money-market instruments. Because it is a secured rate, a repo rate is theoretically very slightly below the corresponding fed funds rate.

**Example (one-day repo):** A corporation invests \$100 million overnight at a repo rate of 5.45%. The repurchase price is:

$$\$100{,}000{,}000 \times \left(1 + \frac{0.0545}{360}\right) = \$100{,}015{,}139$$

The corporation has effectively made a one-day loan at 5.45%, earning \$15,139 in interest.

For a **term repo** of seven days at the same rate:

$$\$100{,}000{,}000 \times \left(1 + \frac{7 \times 0.0545}{360}\right) = \$100{,}105{,}972$$

**Unit check:** Rate $r$ is per year, $d/360$ is in years, so $r \times d/360$ is dimensionless. Multiplying by dollars gives dollars.

**Sanity checks:**
- If $d = 0$, then $L_1 = L_0$ (no interest for zero days)
- If $r$ increases, $L_1$ increases linearly for fixed $d$ (higher rate = more interest)

### 9.1.3 Overnight and Term Repo

**Overnight repo** starts today and ends the next business day. It is the most common type of repo.

**Term repo** has a maturity beyond overnight (days to months). It allows borrowers and lenders of cash to lock in a rate over the term.

> **Desk Reality:** Funding a position with overnight repo means you must roll the trade. Your realized financing P&L depends on both the level of the overnight rate and whether funding remains available on the terms you assumed.

### 9.1.4 SOFR (Orientation)

The secured overnight financing rate (SOFR) is an important volume-weighted median average of the rates on overnight repo transactions in the United States.

---

## 9.2 Financing Long Positions: Selling the Repo

### 9.2.1 The Dealer's Problem

When a trading desk buys bonds from a client, it needs cash to pay for them. Rather than using the institution's capital, the desk will often **repo out** the securities (also called "sell the repo"): deliver the bonds as collateral, receive cash, and agree to repurchase later.

The terminology matters because it indicates the economic position:
- **Sell the repo** = deliver securities, receive cash, agree to repurchase later
- You are the **cash borrower** and **securities lender**

### 9.2.2 A Complete Example

**Worked example:** the following concrete numbers illustrate the full mechanics. Assume:
- Trade date: February 14, 2001; settlement: February 15, 2001
- Security: \$100 million face of the 5-7/8s of November 15, 2005
- Bid price: 103-18 (= 103.5625)
- Accrued interest: 1.493094 (92 days into a 181-day coupon period)

The invoice price due to the seller (a mutual fund) is:

$$\text{Invoice price per 100} = 103.5625 + 1.493094 = 105.055594$$

so the invoice cash for \$100 million face is:

$$\$100{,}000{,}000 \times \frac{105.055594}{100} = \$105{,}055{,}594$$

The trading desk borrows this amount overnight at the market repo rate of 5.10%. On February 16, the desk owes:

$$\$105{,}055{,}594 \times \left(1 + \frac{0.051}{360}\right) = \$105{,}055{,}594 + \$14{,}883 = \$105{,}070{,}477$$

**The cost of financing the overnight position is \$14,883.**

If no buyer emerges the next day, the desk must **roll** its repo—either extending the existing agreement or replacing it with financing from another cash investor.

### 9.2.3 Mapping Cash to Invoice Price

The cash borrowed in repo is typically set equal to (or close to) the **invoice/dirty cash settlement amount** of the collateral. If clean price \(P(0)\) and accrued interest \(AI(0)\) are quoted **per 100** of face, define the invoice price per 100 as \(I(0)=P(0)+AI(0)\). Then:

$$\text{Invoice cash} = \frac{N}{100} I(0) = \frac{N}{100}\left(P(0) + AI(0)\right)$$

In a stylized "no-haircut" repo, \(L_0 \approx \text{Invoice cash}\). With a haircut \(h\), \(L_0=(1-h)\times \text{Invoice cash}\).

> **Pitfall — Clean vs invoice (dirty) funding:** Repo cash is linked to the **invoice** price \(I=P+AI\), not the clean quote \(P\).
> **Why it matters:** Using clean instead of invoice under-funds the purchase and breaks carry/forward calculations.
> **Quick check:** For \$100mm face, 1.00 of accrued interest (per 100) is \$1.0mm of settlement cash.

---

## 9.3 Shorting Securities: Buying the Repo (Reverse Repo)

### 9.3.1 The Mechanics of a Short Sale

A trading desk that sells bonds it doesn't own—going **short**—must deliver those bonds to the buyer. The solution is a **reverse repurchase agreement**: the desk lends cash to someone who owns the bonds, takes the bonds as collateral, and delivers them to the buyer.

The terminology reflects the mirror-image nature of the transaction:
- **Repo (sell the repo):** You deliver securities, receive cash, and agree to repurchase. You are the cash borrower.
- **Reverse repo (buy the repo):** You deliver cash, receive securities, and agree to resell. You are the cash lender—but your motivation is to obtain specific securities.

Repo and reverse repo are the same contract viewed from opposite sides. The term **reverse repo** is useful because it highlights the security-borrowing motivation of the cash lender.

### 9.3.2 Covering a Short

The purchase and delivery of securities that had been sold and borrowed is called **covering a short**. If the desk cannot find a seller, it must **roll its short**—extending the repo or finding another counterparty willing to lend the bonds.

### 9.3.3 Net Borrowers of Cash

Dealers use repo to finance inventories. Reverse repo is just the other side of the same market and is the standard way to borrow a particular security when running a short.

---

## 9.4 Carry: Decomposing Funded Bond P&L

### 9.4.1 The Definition of Carry

On a desk, it is common to split P&L into (i) price moves and (ii) carry. In this chapter, **carry** means the interest earned while you hold the position minus the cost of funding it.

Let:
- $N$: Face amount (notional)
- $P(0), P(d)$: Clean prices **per 100** of face at initiation and $d$ days later
- $AI(0), AI(d)$: Accrued interest **per 100** of face at initiation and $d$ days later
- $I(0)=P(0)+AI(0)$: Invoice (dirty) price **per 100** at initiation
- $r$: Repo rate
- $c$: Coupon rate
- $D$: Actual days in the coupon period

Assume for the moment that there are no coupon payments over the term. In the simplest no-haircut case (financing the full invoice amount), the P&L **per 100** from buying the bond and selling it $d$ days later can be written as:

$$\boxed{\text{P\&L}_{100} = P(d)+AI(d) - (P(0)+AI(0))\left(1+r\frac{d}{360}\right)}$$

Expanding:

$$\boxed{\text{P\&L}_{100} = \underbrace{(P(d)-P(0))}_{\text{Price change}} + \underbrace{(AI(d)-AI(0))}_{\text{Interest income}} - \underbrace{(P(0)+AI(0))\left(r\frac{d}{360}\right)}_{\text{Financing cost}}}$$

So, per 100,

$$\boxed{\text{Carry}_{100} = (AI(d)-AI(0)) - (P(0)+AI(0))\left(r\frac{d}{360}\right)}$$

To convert from "per 100" to currency P&L for face amount $N$, multiply by $N/100$.

> **Pitfall — "per 100" vs "%" vs decimals:** Bond prices and accrued interest are usually quoted **per 100** of face (price "points"), while rates like repo are quoted in **% per year**, and \(1\text{ bp}=10^{-4}\).
> **Why it matters:** Mixing these units is a common way to be off by 100× or 10,000× in carry/financing P&L.
> **Quick check:** If \(I(0)=102.90\) per 100 and \(N=\$100\text{mm}\), invoice cash should be \((N/100)\,I(0)=\$102.9\text{mm}\), not \(\$10.29\text{bn}\).

### 9.4.2 Worked Example (Template): Funding a Long Bond for 7 Days

**Example Title**: Fund a long Treasury for 7 days (carry + `Repo01`)

**Context**
- You buy a Treasury note and fund it in repo for a week.
- You want the expected carry (ignoring price moves) and your sensitivity to the repo rate.

**Timeline (Make Dates Concrete)**
- Trade date: 2026-02-02
- Settlement / repo start: 2026-02-03
- Repo end: 2026-02-10 (7 calendar days; 360-day basis)
- Coupon: assume no coupon payment inside the 7-day horizon

**Inputs**
- Face amount: \(N=\$100{,}000{,}000\)
- Clean price quote: \(P(0)=102\text{-}16 = 102.50\) (per 100)
- Accrued interest at settlement: \(AI(0)=0.40\) (per 100) so invoice \(I(0)=P(0)+AI(0)=102.90\)
- Repo rate: \(r=4.80\%\) (simple; 360-day basis), \(d=7\)
- Haircut: \(h=2\%\) (cash borrowed is \((1-h)\times\) invoice cash)
- Coupon for accrual illustration: assume coupon rate \(c=6\%\) (semiannual coupons) and current coupon period length \(D=182\) days

**Outputs (What You Produce)**
- Repo start cash \(L_0\), repo interest, coupon accrual, carry (repo-only)
- Funding-rate risk: `Repo01` (bump object + units + sign)

**Step-by-step**
1. Invoice cash at settlement:
   \[
   \text{Invoice cash}=N\times I(0)/100 = 100{,}000{,}000\times 102.90/100 = \$102{,}900{,}000.
   \]
2. Repo start cash with haircut:
   \[
   L_0=(1-h)\times \text{Invoice cash}=0.98\times 102{,}900{,}000=\$100{,}842{,}000.
   \]
   Equity posted \(E_0 = \text{Invoice cash}-L_0 = \$2{,}058{,}000\).
3. Repo interest over 7 days:
   \[
   \text{Repo interest}=L_0 \times r \times d/360 = 100{,}842{,}000 \times 0.048 \times 7/360 = \$94{,}119.20.
   \]
4. Coupon accrual over 7 days (assumption above):
   \[
   \text{Accrual}=N\times (c/2)\times (d/D) = 100{,}000{,}000\times 0.03 \times 7/182 = \$115{,}384.62.
   \]
5. Repo carry (ignoring price changes and ignoring the funding cost of the haircut equity):
   \[
   \text{Carry}\approx \text{Accrual} - \text{Repo interest} = \$21{,}265.42.
   \]
6. Funding-rate risk (`Repo01`):
   Define `Repo01` as \(\text{P\&L}(r-1\text{bp})-\text{P\&L}(r)\) holding \(L_0\) and everything else fixed. With simple interest on a 360-day basis,
   \[
   \text{Repo01}=L_0 \times (d/360) \times 10^{-4} = \$196.08 \text{ per 1bp}.
   \]
   This is **positive** for a funded long: a 1bp drop in the repo rate reduces financing cost.

**Cashflows (table)**
| Date | Cashflow | Explanation |
|---|---:|---|
| 2026-02-03 | \(-\$102{,}900{,}000\) | Pay invoice price to acquire the bond |
| 2026-02-03 | \(+\$100{,}842{,}000\) | Receive repo cash (haircut-adjusted) |
| 2026-02-10 | \(-\$100{,}842{,}000 - \$94{,}119.20\) | Repay repo principal plus repo interest |
| (over 2026-02-03→10) | \(+\$115{,}384.62\) | Accrual P&L from holding the bond (not a cash coupon unless a payment date occurs) |

**P&L / Risk Interpretation**
- `Repo01` is a funding-rate DV01: it tells you how much P&L improves if your repo rate is 1bp lower for the same term and borrowed amount.
- Haircuts reduce the repo interest you pay (because you borrow less), but they also create an unfunded equity portion; your all-in carry depends on how you fund that equity.

**Sanity Checks**
- Units check: \(r\) is per year, \(d/360\) is years, so \(L_0 r d/360\) is currency.
- Sign check: for a funded long, repo rate down \(\Rightarrow\) lower financing cost \(\Rightarrow\) higher P&L, so `Repo01` should be positive.
- Limit check: as \(d\to 0\), both repo interest and `Repo01` go to zero.

### 9.4.3 Quick Numerical Example

Continuing the example of Section 9.2, suppose the desk sells the bond the next day at the ask price (one tick higher than bid). The desk earns \$32,596 on the trade:

- **Price change (bid-ask spread):** $\$100{,}000{,}000 \times (1/32)\% = \$31{,}250$
- **Interest income (one day's accrual):** $\$100{,}000{,}000 \times (1.509323 - 1.493094)\% = \$16{,}229$
- **Financing cost:** $\$14{,}883$ (from Section 9.2)
- **Carry:** $\$16{,}229 - \$14{,}883 = \$1{,}346$
- **Total P&L:** $\$31{,}250 + \$1{,}346 = \$32{,}596$

### 9.4.4 When Is Carry Positive?

In the example above, carry is positive because the bond's coupon rate (5.875%) exceeds the repo rate (5.10%). However, the relationship is not exact:

1. Interest income is earned on **face value**, while financing cost is applied to the **full (invoice) price**
2. Coupon accrual uses the bond's day count, while repo uses a **360-day basis** (\(d/360\))

These differences can cause carry to be slightly different from a naive coupon-minus-repo calculation.

### 9.4.5 Carry and Shorts

For a short position, the signs flip: the desk **pays** the coupon (to the security lender) and **earns** repo on its cash. In the same numerical setup, the short earns only \$29,908, compared to \$32,596 for the long—the difference is the negative carry of approximately \$1,342 (plus rounding).

### 9.4.6 Breakeven Calculations

Carry is useful for computing **breakeven price changes**. In this same setup, an investor holding the 5-7/8s for 30 days at 5.10% repo earns:

- Interest income: $\$486{,}878$
- Financing cost: $\$446{,}688$
- Carry: $\$40{,}190$

The bond price can fall by about 4 cents per 100 face before the investment loses money.

Similarly, for shorts, carry determines **breakeven holding periods**. A short expecting a price fall from 105.055594 to 105 must see the move within about 41 days before negative carry offsets the price gain.

### 9.4.7 Carry Does Not Determine Expected Returns

Positive carry trades "earn money as they go," but that does not imply a higher expected return than a negative-carry trade. Carry is just one component of total return.

A premium bond has positive carry but faces price pull-to-par; a discount bond has negative carry but gains from price appreciation. The expected return of any fairly priced portfolio equals the short-term rate plus a risk premium, whether that return comes from carry or price change. See Chapter 7 for a fuller treatment of return decomposition.

---

## 9.5 General Collateral vs Specials

### 9.5.1 Definitions: GC vs Special Repo

There are repo rates for general collateral (any Treasury security), and there are often special repo rates for a particular Treasury security.

In this book:
- **General collateral (GC)** repo: the cash lender will accept **any** eligible security in a broad set (e.g., "any Treasury"). The GC rate is the prevailing secured funding rate for that eligible set.
- **Special** repo: the trade specifies a **particular** security (a particular CUSIP). Specials arise when there is unusual demand to borrow that security, so the special repo rate is lower than the GC rate.

### 9.5.2 Why GC Tends to Trade Below Unsecured Rates

Because it is a secured rate, a repo rate is theoretically very slightly below the corresponding fed funds rate.

### 9.5.3 The Specialness Spread

Define the **specialness spread**:

$$\boxed{s = r_{\text{GC}} - r_{\text{spec}}}$$

When a bond is special, \(r_{\text{spec}} < r_{\text{GC}}\), so \(s>0\). "Specialness" is the scarcity value of that security, expressed as a funding advantage.

### 9.5.4 A Concrete Snapshot (2001-02-15)

On 2001-02-15, a representative snapshot shows a GC rate of 5.44% and much lower special rates for on-the-run issues:

| Treasury Issue | Comment | Special Rate | Specialness |
|---|---|---:|---:|
| 5.75% Nov 15, 2005 | On-the-run 5-year | 3.85% | 159 bp |
| 5.75% Aug 15, 2010 | On-the-run 10-year | 4.25% | 119 bp |
| 4.75% Jan 31, 2003 | On-the-run 2-year | 4.88% | 56 bp |

The key point is not the exact numbers but the mechanism: on-the-run collateral can become scarce to borrow, pushing its special repo rate well below the GC rate.

### 9.5.5 The Dollar Value of Specialness (Carry Impact)

Specialness matters because it directly changes financing cost. Over \(d\) days, the difference between financing at GC and financing at a special rate is approximately:

$$\text{Financing difference} \approx L_0 \times s \times \frac{d}{360}$$

For example, if \(L_0=\$100{,}000{,}000\), \(s=159\) bp, and \(d=1\),

$$\text{Daily benefit} \approx \$100{,}000{,}000 \times \frac{0.0159}{360} \approx \$4{,}417.$$

For a short position, specialness shows up as a **cost**: to borrow the security, you are effectively lending cash at the low special rate instead of at GC.

---

## 9.6 Special Repo Rates and the Auction Cycle

### 9.6.1 The Pattern of On-the-Run Specialness

A common pattern in on-the-run (OTR) specialness over the auction cycle: specialness tends to be smaller right after an auction or reopening (more supply) and tends to build as the issue becomes the focal point for shorting and financing demand.

**Immediately after an auction:** The new on-the-run issue has just been distributed. Shorts can stay in the previous on-the-run or shift to the new one—this substitutability depresses special spreads. After a **reopening** (an auction that increases the size of an existing issue), extra supply also depresses spreads.

**As time passes:** Shorts often migrate toward the on-the-run, and its specialness can rise.

### 9.6.2 Volatility and Magnitude

Specialness is a supply/demand price for collateral, so it can move sharply day-to-day. In stressed or scarce-collateral episodes, specialness can become very large (hundreds of basis points).

### 9.6.3 The Floor on Special Rates: The Pre-2009 Logic

A useful “borrow vs fail” argument for a floor on special repo rates: if the bond cannot be borrowed, the trader will fail to deliver it and, consequently, not receive the proceeds from the sale. In effect, the trader will lose one day of interest on the proceeds.

If the bond can be borrowed, the trader delivers the bond, receives the proceeds, and lends them at the special repo rate. But if the repo rate is 0%, there is no point in bothering with the repo agreement: earning 0% on the proceeds is the equivalent of having failed to deliver the bond.

Therefore, the special rate cannot fall below 0%, and, equivalently, the special spread cannot be greater than the GC rate. Under this opportunity-cost-only logic:

$$r_{\text{spec}} \geq 0\% \quad \Rightarrow \quad s \leq r_{\text{GC}}$$

In the fall of 2001, with GC near 2%, the maximum special spread was about 200 basis points.

---

## 9.7 Valuing a Bond Trading Special in Repo

### 9.7.1 The Relative Value Question

An important application: how should an investor choose between two similar bonds when one trades special? Consider a money manager deciding between the on-the-run 5-year (yielding 4.970%) and an off-the-run issue with the same maturity (yielding 5.020%). The OTR is five basis points rich—but it also finances at 159 basis points below GC.

### 9.7.2 Components of the Relative Value

One useful framework breaks the decision into three components:

1. **Liquidity value**: How much is the OTR's superior liquidity worth to this investor? A trader who expects to sell quickly values liquidity highly; a buy-and-hold investor values it less.

2. **Financing advantage**: If the OTR is expected to trade 100 bp special on average over a 90-day horizon, the financing benefit is approximately:

$$\frac{0.01 \times 90}{360} = 0.25\%$$

of face value.

3. **Yield rolldown**: If the OTR's yield premium is expected to narrow from 5 bp to 3 bp over the horizon, this creates a price loss as the OTR converges toward fair value.

### 9.7.3 The Trade-Off Calculation

Netting these effects: if the financing advantage is 0.25% of face, but the anticipated yield convergence costs about 0.085% (based on DV01), and the coupon disadvantage costs about 0.031%, then the net advantage of the OTR is approximately:

$$0.25\% - 0.031\% - 0.085\% = 0.134\%$$

This is equivalent to about 3.1 basis points. But if the OTR trades 4 basis points rich (after accounting for 1 bp of liquidity value), the off-the-run is the better investment for this particular investor.

**Key insight:** Any trade or hedge involving a security that is trading special requires the same set of assumptions and calculations. How much is liquidity worth? How will special spreads behave? How will the premium change over time?

---

## 9.8 Haircuts, Repricing, and Leverage

### 9.8.1 Why Haircuts Exist

What if the collateral value was to suddenly drop? This would lead to losses for the collateral buyer. One way to protect against such potential losses would be for the collateral buyer to pay less than the market value of collateral to the seller. The difference between the market value of the collateral and the amount paid by the buyer to the seller is called the haircut. It is a way of imposing a margin on the collateral seller similar to that of initial margin in futures markets.

### 9.8.2 Haircut Mechanics

Under one common convention, the haircut $h$ is a percentage reduction of collateral market value:

$$\boxed{L_0 = (1 - h) \times \frac{N}{100} \times I(0)}$$

where \(\frac{N}{100}I(0)\) is the invoice (dirty) market value of the collateral in currency units (when \(I(0)\) is quoted per 100 of face).

Always verify how haircuts are defined and quoted in your documentation.

### 9.8.3 Equity and Leverage

The equity (unfunded portion) posted by the repo borrower is:

$$E_0 = \frac{N}{100} \times I(0) - L_0 = h \times \frac{N}{100} \times I(0)$$

This implies leverage of:

$$\boxed{\text{Leverage} = \frac{(N/100)\, I(0)}{E_0} = \frac{1}{h}}$$

**Example:** A 2% haircut implies 50× leverage. A 5% haircut implies 20× leverage.

### 9.8.4 Margin Calls and Repricing

If collateral value falls, the loan may exceed the permitted amount under the haircut. The borrower must either post additional collateral or repay cash—this is a **margin call**.

**Example:** Collateral market value falls from \$103 million to \$101 million with a 2% haircut. Maximum permitted loan is \(0.98 \times 101 = \$98.98\) million. If the outstanding loan is \$100.94 million, the margin call is \$1.96 million.

This creates **liquidity risk**: price declines generate immediate cash demands. In stress scenarios, inability to meet margin calls can force liquidation at unfavorable prices.

> **Desk Reality:** Repricing turns market moves into *immediate funding needs*. Even if your position is "right" in the long run, you can be forced to delever if you cannot meet margin calls on time.

**Worked Example: Forced Deleveraging from Haircut Increase**

A fund has \$100 million in bonds, financed with 2% haircut (\$98mm loan, \$2mm equity).

Haircut increases to 5%. Maximum loan is now \$95mm. The fund must:
- Repay \$3mm of the loan, OR
- Sell \$3mm / 0.95 = \$3.16mm of bonds to stay within the new haircut

If the fund lacks \$3mm cash, it must sell—potentially at distressed prices.

---

## 9.9 Repo as a Forward-Pricing Link

### 9.9.1 The Replication Argument

A fundamental insight is that buying a bond spot and financing it through repo replicates a **forward position** in the bond: you lock in (most of) the economics of owning the bond over the horizon via the financing rate.

### 9.9.2 Forward Invoice Price (No Intermediate Coupon)

Under no-arbitrage, the forward invoice price equals the terminal value of the repo loan:

$$\boxed{P_{\text{fwd}} + AI(d) = (P(0) + AI(0)) \times \left(1 + r \frac{d}{360}\right)}$$

Rearranging:

$$P_{\text{fwd}} = (P(0) + AI(0)) \times \left(1 + r \frac{d}{360}\right) - AI(d)$$

Or equivalently:

$$\boxed{P_{\text{fwd}} = P(0) - \text{Carry}}$$

Positive carry generally implies a forward price below the spot price (for coupon bonds).

### 9.9.3 Forward Price with an Intermediate Coupon

If a coupon is paid during the forward period, the analysis becomes more complex. The bond owner continues to receive the coupon; the repo borrower must pass it through (a "manufactured coupon"). Economically, the repo loan balance is reduced by the coupon amount.

Let:
- $d_1$ = days from initiation to coupon date
- $d_2$ = days from coupon date to delivery
- $C$ = coupon payment (e.g., $c/2$ for a semiannual bond)

Then:

$$\boxed{P_{\text{fwd}} + AI(d) = \left[(P(0) + AI(0))\left(1 + r\frac{d_1}{360}\right) - C\right]\left(1 + r\frac{d_2}{360}\right)}$$

The security lender continues to receive coupons, so the borrower must pass through a "manufactured" coupon to the lender.

Intuition: the security lender continues to receive the coupon, so the borrower must pass through a manufactured coupon; it is therefore natural to reduce the repo loan balance by the coupon on the coupon date.

**Sanity check:** If $C = 0$, this collapses to the no-coupon formula.

### 9.9.4 Implied Repo Rate

The relationship can be inverted. Given observed spot and forward prices, the **implied repo rate** is the financing rate that makes the arbitrage hold:

$$\boxed{r_{\text{implied}} = \frac{P_{\text{fwd}} + AI(d) - (P(0) + AI(0))}{(P(0) + AI(0))} \times \frac{360}{d}}$$

**Interpretation:** The implied repo rate is the financing cost "baked into" the forward price. If implied repo exceeds actual repo, the forward is cheap—you can buy spot, finance at actual repo, and sell forward for a profit.

**Worked Example: Computing Implied Repo**

- Spot invoice \(P(0)+AI(0)=102.50\)
- Forward invoice \(P_{\text{fwd}}+AI(d)=102.75\)
- Days to delivery \(d=30\)

$$r_{\text{implied}} = \frac{102.75 - 102.50}{102.50} \times \frac{360}{30} = 0.00244 \times 12 = 2.93\%$$

If your actual repo is 3.00%, this forward is slightly "rich" relative to cash-and-carry (it embeds cheaper financing than you actually face).

### 9.9.5 Preview: The Basis Trade

The implied repo concept is central to Treasury futures "basis" logic: futures prices embed an implied financing rate for a deliverable bond. Comparing that implied rate to your actual financing rate is a core input into cash-and-carry trade evaluation (developed in detail in Chapter 23).

---

## 9.10 Market Stress: September 11, 2001

### 9.10.1 Disruption in the Specials Market

The September 11, 2001 attacks disrupted the Treasury specials market through two channels: operational disruption that increased settlement fails, and a pullback of securities lending amid uncertainty and credit concerns. The combination created a severe shortage of on-the-run collateral.

### 9.10.2 Extreme Special Rates

On September 20, 2001, repo rates reached extraordinary levels:

| Treasury Issue | Comment | Special Rate |
|---------------|---------|--------------|
| GC rate | | 1.75% |
| 4.625% May 15, 2006 | OTR 5-year | 0.10% |
| 5.000% Aug 15, 2011 | OTR 10-year | 0.35% |
| 3.625% Aug 31, 2003 | OTR 2-year | 0.65% |

In this episode, GC was about 1.75% while the fed funds rate at the time was about 3%, widening the spread between fed funds and GC to about 125 basis points.

The OTR 5-year was 165 basis points special—nearly the entire GC rate.

### 9.10.3 The Treasury's Response

On October 4, 2001, the Treasury announced a surprise same-day auction of \$6 billion of the on-the-run 10-year, which was unusual both because issuance is typically scheduled and because the market normally receives more notice.

The response was immediate: 10-year futures prices fell sharply, the spread between OTR and old 10-year notes compressed from 5.2 to 3.7 basis points, and the OTR 10-year special spread fell about 100 basis points.

### 9.10.4 Lesson: Operational Frictions Matter

The September 2001 episode illustrates that repo markets can become severely stressed when collateral supply is disrupted. Fails, credit concerns, and collateral hoarding can drive special rates to their floor and dislocate relative value relationships. Understanding these dynamics is essential for risk management.

---

## 9.11 Practical Notes

### 9.11.1 Market Structure

A robust clearing/custody/settlement infrastructure is an essential prerequisite for a well-functioning repo market. At a high level, repo is often discussed in two structures: **bilateral** and **tri-party**.

For this chapter, the key operational distinction is:

| Structure | High-level idea | Why it matters here |
|---|---|---|
| Bilateral repo | Two parties manage collateral selection and settlement directly | Common framing for **security-specific** ("special") collateral discussions |
| Tri-party repo | A third-party agent helps manage collateral allocation, valuation, and settlement within an eligibility set | Useful intuition for **GC-style** funding where collateral is a basket |

### 9.11.2 Common Quoting and Contract Gotchas

| Issue | Notes |
|-------|-------|
| **Rate convention** | Repo uses simple interest on a 360-day basis (\(d/360\)) in this chapter. Bond accrual may use different day counts. |
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
| **Cashflow reconciliation** | Initial cash + interest - coupon adjustments = repurchase cash |
| **Special rate reasonableness** | Specials can be very low. If you see extreme specials, sanity-check against "borrow vs fail" economics and your sign conventions |
| **Leverage check** | With haircut $h$, leverage should be near $1/h$ |
| **Sensitivity sign** | Higher repo rate → higher financing cost → $\partial \text{P\&L}/\partial r < 0$ for funded longs |
| **`Repo01` sign** | With `Repo01 := \text{P\&L}(r-1\text{bp})-\text{P\&L}(r)`, a funded long should have `Repo01 > 0` |

---

## Summary

1. **Repo is secured short-term funding**: Cash versus collateral today, reversed at a higher repurchase price embedding repo interest.

2. **Repo vs reverse repo** are the same transaction from opposite perspectives. Sign conventions matter for P&L.

3. **Dealers use repo** to finance long inventory and execute short sales without tying up balance sheet capital.

4. **GC repo** reflects general secured funding; **specials** reflect demand for particular securities, often trading at lower rates.

5. **Specialness** ($r_{\text{GC}} - r_{\text{spec}}$) measures scarcity value and has real dollar impact via financing costs.

6. **Borrow vs fail bounds specials:** under an opportunity-cost-only framing, special rates cannot fall below 0%, so specialness cannot exceed the GC rate.

7. **Haircuts** create overcollateralization and limit leverage; repricing creates margin call liquidity risk.

8. **Carry** ≈ coupon accrual - repo interest (financing cost). Positive carry when coupon rate exceeds repo rate (approximately).

9. **Repo links spot and forward prices**: buying spot and financing at repo replicates a forward position.

10. **Implied repo** reveals the financing rate embedded in forward/futures prices—key to basis trading.

11. **The secured overnight financing rate (SOFR)** is an important volume-weighted median average of the rates on overnight repo transactions in the United States.

12. **In stress**, collateral shortages can drive specials to extreme levels and dislocate relative value.

---

## Key Concepts

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| Repo | Sale and repurchase agreement; economically a secured loan | Core funding mechanism for fixed income |
| Reverse repo | Same as repo from cash lender's view | Used to borrow specific securities |
| GC rate | Rate for repos accepting any eligible collateral | Benchmark secured funding rate; feeds into SOFR |
| Special rate | Rate for repos requiring specific collateral | Reflects scarcity/short demand |
| Specialness spread | $r_{\text{GC}} - r_{\text{spec}}$ | Dollar value of owning special bonds |
| Haircut | Reduction in collateral value for lending | Controls leverage, protects lender |
| Carry | Interest income minus financing cost | Key component of funded P&L |
| `Repo01` | Funding-rate sensitivity: \(\text{P\&L}(r-1\text{bp})-\text{P\&L}(r)\) holding \(L_0,d\) fixed (currency per 1bp) | Makes funding risk explicit (separate from yield DV01) |
| Implied repo | Financing rate implied by spot/forward prices | Reveals embedded financing in positions |
| SOFR | The secured overnight financing rate (SOFR) is an important volume-weighted median average of the rates on overnight repo transactions in the United States | Based on overnight repo transactions |
| Settlement fail | Failure to deliver a security on settlement | Creates "borrow vs fail" economics for specials and shorts |

---

## Notation

| Symbol | Meaning | Units / Convention |
|---|---|---|
| \(N\) | Face amount (notional) | currency |
| \(P\) | Clean price | points per 100 of face |
| \(AI\) | Accrued interest | points per 100 of face |
| \(I=P+AI\) | Invoice (dirty) price | points per 100 of face |
| \(L_0\) | Repo start cash (cash borrowed/lent at start leg) | currency; in this chapter \(L_0 \approx (1-h)\,(N/100)\,I(0)\) |
| \(L_1\) | Repo end cash (repurchase cash) | currency; \(L_1=L_0(1+r\,d/360)\) in the simple 360-day convention |
| \(r\) | Repo rate | per year; simple interest; 360-day basis in examples |
| \(d\) | Repo term length | days |
| \(h\) | Haircut | unitless fraction; \(L_0=(1-h)\times\) collateral market value |
| \(r_{\\text{GC}}\) | General collateral repo rate | per year |
| \(r_{\\text{spec}}\) | Special repo rate (specific CUSIP) | per year |
| \(s=r_{\\text{GC}}-r_{\\text{spec}}\) | Specialness spread | per year (or bp); positive when collateral is "special" |
| \(r_{\\text{implied}}\) | Implied repo rate | per year; backed out from spot/forward (or futures) pricing |
| `Repo01` | Funding-rate sensitivity for a funded position | currency per 1bp; defined as \(\text{P\\&L}(r-1\\text{bp})-\\text{P\\&L}(r)\) holding \(L_0\) and \(d\) fixed |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is a repo economically? | A secured loan: borrower posts securities as collateral and repays cash plus interest |
| 2 | What is a reverse repo? | The cash lender's side of a repo; used to borrow specific securities |
| 3 | What are the two legs of a repo? | Start leg (cash vs collateral) and end leg (repurchase at higher price) |
| 4 | Why is the repurchase price higher? | It embeds repo interest |
| 5 | Write the repurchase formula | $L_1 = L_0(1 + rd/360)$ under simple interest on a 360-day basis |
| 6 | What is the GC rate? | The repo rate for general (non-specific) Treasury collateral |
| 7 | What is a special repo? | A repo for a specific security, often at below-GC rates |
| 8 | Define specialness spread | $s = r_{\text{GC}} - r_{\text{spec}}$ |
| 9 | Why do on-the-run issues trade special? | High demand to short them for liquidity reasons |
| 10 | How do you calculate the dollar value of specialness? | $L_0 \times s \times d/360$ |
| 11 | What is a haircut? | A reduction in collateral value that creates overcollateralization |
| 12 | How does haircut relate to leverage? | Leverage ≈ 1/h |
| 13 | What is carry (funded position)? | Interest income minus financing cost |
| 14 | When is carry typically positive? | When coupon rate exceeds repo rate |
| 15 | Write the funded P&L formula (no coupon payment in the horizon) | $\text{P\&L}=\frac{N}{100}[P(d)-P(0)+AI(d)-AI(0)]-L_0\,r\,d/360$ |
| 16 | How does repo link to forward prices? | Forward invoice = spot invoice × (1 + rd/360), so $P_{\text{fwd}} = P(0) - \text{Carry}$ |
| 17 | What can limit how negative a special can trade? | Under a "borrow vs fail" opportunity-cost framing, special rates cannot fall below 0%, so specialness cannot exceed the GC rate |
| 18 | What is a manufactured coupon? | The coupon payment passed from security borrower to lender during a repo |
| 19 | What is repo-rate risk? | P&L sensitivity to changes in financing rates |
| 20 | What is SOFR and why does it matter? | The secured overnight financing rate (SOFR) is an important volume-weighted median average of the rates on overnight repo transactions in the United States |
| 21 | What is implied repo? | The financing rate embedded in the forward-spot price relationship |
| 22 | What does high specialness signal? | Crowded short interest—potential squeeze risk |

---

## Mini Problem Set

1. (Compute) A repo starts with \(L_0=\$25{,}000{,}000\), \(r=3.75\%\), \(d=3\) days (360-day basis). Compute \(L_1\) and the interest.
2. (Compute) A bond has clean price 99.75 and accrued interest 1.10 (both per 100 notional). For \(N=\$50{,}000{,}000\) face, compute the invoice cash exchanged.
3. (Concept) When is carry positive for a funded long? Name two convention effects that can make "coupon - repo" misleading.
4. (Compute) For \(N=\$100{,}000{,}000\), invoice price 102% of face, and \(d=7\), compute the P&L change for a +10bp move in the repo rate (hold everything else fixed).
5. (Compute) Collateral market value is \(\$80{,}000{,}000\); haircut is 5%. Compute cash lent and leverage.
6. (Compute) Using Problem 5, if collateral falls to \(\$76{,}000{,}000\), compute the margin call.
7. (Compute) \(r_{\text{GC}}=4.5\%\), \(r_{\text{spec}}=0.5\%\), \(L_0=\$200{,}000{,}000\), \(d=1\). Compute the daily specialness benefit.
8. (Compute/Desk) Build a 40-day repo spanning a coupon. Show cashflows under a stated coupon/loan adjustment convention.
9. (Derive) Given spot invoice \(I_0\) and repo rate \(r\), derive the forward clean price formula.
10. (Derive) Repeat Problem 9 with one coupon inside the forward horizon.
11. (Desk) Describe two distinct risks when funding a term trade by rolling overnight repo.
12. (Desk) A bond becomes more special after you short it. How does that affect your P&L and funding?

### Solution Sketches (Selected)
1. \(L_1=L_0(1+r d/360)=25{,}000{,}000(1+0.0375\times 3/360)=\$25{,}007{,}812.50\). Interest \(=\$7{,}812.50\).
2. Invoice per 100 \(=99.75+1.10=100.85\). Cash \(=50{,}000{,}000\times 100.85/100=\$50{,}425{,}000\).
3. Carry is positive when coupon accrual exceeds financing cost. Two common "gotchas": financing is applied to invoice (not face), and coupon accrual day count differs from the repo 360-day basis.
4. Financing cost change \(\approx -(N\times 1.02)\times (7/360)\times 0.0010=-\$1{,}983.33\).
5. Cash lent \(L_0=0.95\times 80{,}000{,}000=\$76{,}000{,}000\). Equity \(=\$4{,}000{,}000\). Leverage \(=1/0.05=20\times\).
6. Max permitted loan \(=0.95\times 76{,}000{,}000=\$72{,}200{,}000\). Margin call \(=76{,}000{,}000-72{,}200{,}000=\$3{,}800{,}000\).
7. Daily benefit \(=L_0(r_{\text{GC}}-r_{\text{spec}})/360=200{,}000{,}000\times 0.04/360=\$22{,}222.22\).

---

## References

- Bruce Tuckman, *Fixed Income Securities* ("Reverse Repurchase Agreements and Short Positions"; "General Collateral and Specials"; "Special Repo Rates and the Auction Cycle"; "Forward Price of a Deposit or a Zero Coupon Bond").
- Salih N. Neftci, *Principles of Financial Engineering* ("Special Versus General Collateral"; "Classic Repo"; "Haircut"; "Implied Repo Rate").
- John C. Hull, *Options, Futures, and Other Derivatives* ("Hedging an Equity Portfolio" → repo and SOFR overview).
- Robert A. Jarrow, *Modeling Fixed Income Securities and Interest Rate Options* ("Traded Securities > Treasury Security Markets" → repo as a forward contract).
- Edwin J. Elton, Martin J. Gruber, Stephen J. Brown, William N. Goetzmann, *Modern Portfolio Theory and Investment Analysis* ("Money Market Securities" → repurchase agreements).
- Damiano Brigo, Massimo Morini, Andrea Pallavicini, *Counterparty Credit Risk, Collateral and Funding* (2013, "Definition of Exposures" → haircuts on non-cash collateral).
- Federal Reserve Bank of New York, "Secured Overnight Financing Rate (SOFR)", accessed 2026-02-15.
