# Chapter 40: The CDS Auction Process — What It Does and Why It Exists

## Learning Objectives

- Explain why auction settlement exists (notional > deliverables and short squeezes).
- Define the auction objects: Deliverable Obligations, Auction Final Price, Initial Market Midpoint (IMM), and Open Interest (OI).
- Translate an Auction Final Price quote into a cash settlement amount, including accrued premium on default.
- Work a full cash-settlement timeline with unit/sign checks.
- State a simple risk measure: sensitivity of protection payout to a +1-point change in Auction Final Price.

**Prerequisites:** [Chapter 38](chapters/chapter_38_cds_contract_mechanics.md), [Chapter 39](chapters/chapter_39_cds_credit_events_settlement.md), [Chapter 35](chapters/chapter_35_default_recovery_credit_events.md)  
**Follow-on:** [Chapter 41](chapters/chapter_41_cds_indices_mechanics_coupons_rolls.md), [Chapter 43](chapters/chapter_43_cds_risks_hedging.md), [Chapter 44](chapters/chapter_44_cds_relative_value_trading_frameworks.md)

## Introduction

A CDS credit event forces the market to answer a practical question: *at what price do we settle all outstanding protection on the name?* Physical settlement answers by delivering bonds; cash settlement answers by paying a cash amount linked to a market price of deliverables.

When CDS notional is large relative to deliverable debt, forcing everyone into physical settlement can create a scramble for a scarce set of deliverable bonds (a short squeeze). Settlement demand then distorts bond prices and the operational burden of sourcing deliverables becomes the limiting factor. The ISDA auction mechanism was designed to produce a single **Auction Final Price** for the deliverable obligations, which then maps mechanically into a cash settlement amount for every contract on the same reference entity.

A widely cited stress test is the Lehman Brothers bankruptcy (September 2008): estimates put CDS notional far larger than the amount of deliverable Lehman obligations, and the auction produced an Auction Final Price of 8.625 (per 100), implying a cash settlement of 91.375% of notional for long protection.

This chapter focuses on three links in the chain:

- **Quote:** the auction output $FP_{100}$ (Auction Final Price, points per 100 outstanding principal).
- **Cashflows:** protection payout and the (often forgotten) accrued premium owed up to the event date.
- **Risk:** sensitivity of the payout to the auction final price and the basis risk created by deliverable dispersion.

## 40.1 The Problem: When CDS Notional Exceeds Debt Outstanding

### 40.1.1 Physical Settlement and Scarcity

Chapter 39 established that physical settlement requires the protection buyer to deliver eligible obligations with a specified face amount and receive par in cash.

This works well when the amount of CDS protection outstanding is modest relative to the available debt. But the credit derivatives market grew rapidly through the 2000s, and a structural problem emerged: CDS notional outstanding could far exceed the supply of deliverable obligations.

Lehman is a stark example: widely cited estimates put outstanding CDS notional around $400 billion versus roughly $155 billion of deliverable Lehman debt. If everyone tried to source deliverables, the combined buying pressure would create a short squeeze—pushing bond prices up and reducing the effective protection payout under physical settlement.

Mechanically, if protection buyers must buy bonds to deliver, that forced demand can lift post-event bond prices above “fundamental” levels. Because the physical-settlement gain is $N - \text{(cost to acquire deliverables)}$, higher deliverable prices translate into smaller protection gains.

### 40.1.2 The 2005 ISDA Protocol

One market response (documented in the credit-derivatives literature) was an ISDA protocol introduced in 2005: when outstanding derivative notional can exceed the supply of deliverable obligations, contracts can fall back to **auction-based cash settlement**, with the auction determining a single cash settlement price.

This was a pivotal development. The 2005 protocol established that when physical settlement was impractical due to supply constraints, contracts could settle via cash payment based on an auction-determined price. The auction would produce a single price representing the market value of deliverable obligations, and all contracts would settle against that price.

The economic logic is compelling:

1. **Avoids short squeezes:** Protection buyers don't need to source scarce deliverables, eliminating artificial buying pressure that would distort prices
2. **Standardization:** A single auction price applies to all contracts on the same reference entity, rather than forcing bilateral negotiations
3. **Operational simplicity:** Removes the logistics of sourcing, verifying, and delivering physical obligations

> **Analogy: The Market Clearing**
>
> Imagine 100 people promised to deliver a specific rare painting (the bond). There are only 5 paintings in existence.
>
> *   **Physical Settlement Madness**: 100 people fight over 5 paintings. The price skyrockets to USD 100 million. The protection sellers lose everything.
> *   **The Auction Solution**: Everyone agrees to meet in a room. We ask the 5 owners: "What is a fair price to sell?" We ask the 100 buyers: "What is a fair price to settle?"
> *   **Result**: We agree on a single price (e.g., USD 5 million). The 95 people who can't find a painting just pay the cash difference. Order is restored.

---

## 40.2 What the Auction Determines

### 40.2.1 The Final Price Concept

The auction produces an **Auction Final Price** $FP_{100}$ that is used to compute cash settlement payoffs. In ISDA language, this price is expressed as a percentage (points per 100) of the outstanding principal balance of deliverable obligations. In textbook terms, it is often described as a “mid-market” value of deliverables, with an important connection to cheapest-to-deliver (CTD) economics under physical settlement.

This description reveals two important features:

**Mid-market value:** The auction seeks to establish a fair market price, not a distressed fire-sale price or an artificially inflated price driven by settlement demand. The mid-market concept aims for the price at which willing buyers and sellers would transact in normal conditions.

**Cheapest deliverable:** The auction price relates to the cheapest-to-deliver (CTD) concept from Chapter 39. Under physical settlement, rational protection buyers would deliver the lowest-priced eligible obligation. The auction price should reflect this CTD value to maintain economic equivalence between settlement methods.

### 40.2.2 Connection to Settlement Economics

Once the auction establishes $FP_{100}$, cash settlement follows mechanically: the protection seller pays the protection buyer the loss fraction implied by the auction price.

Using our notation where $FP_{100}$ is points per 100 outstanding principal:

$$\boxed{\text{Protection Payout} = N \times \left(1 - \frac{FP_{100}}{100}\right) = N(1-R)}$$

where $R = FP_{100}/100$ is the recovery fraction.

**Lehman example:** The auction determined $FP_{100} = 8.625$, implying $R = 0.08625$. For a protection buyer with USD 100 million notional:

$$\text{Payout} = USD\ 100\text{mm} \times (1 - 0.08625) = USD\ 91.375\text{mm}$$

The key point is not the historical number itself, but the mapping: **auction price $\rightarrow$ settlement cash amount**. Any accrued premium due up to the event date is handled separately (Section 40.5.4).

### 40.2.3 Why Physical and Cash Settlement Should Align

When does cash settlement deliver the same economic outcome as physical settlement? Chapter 39 derived that physical settlement payoff equals $N(1 - P_{CTD}/100)$, where $P_{CTD}$ is the cheapest deliverable price.

**Alignment condition:** If the auction final price equals the CTD market price—that is, if $FP_{100} = P_{CTD}$—then:

$$\Pi_{\text{cash}} = N\left(1 - \frac{FP_{100}}{100}\right) = N\left(1 - \frac{P_{CTD}}{100}\right) = \Pi_{\text{phys}}$$

This is a useful mental model: the auction summarizes the deliverable set into a single settlement price that is intended to be consistent with CTD economics, so that cash settlement approximates the economic result of delivering the cheapest eligible obligation.

---

## 40.3 The Auction Mechanism

The high-level story in the textbooks (“two-stage auction”, “mid-market value of deliverables”) becomes precise in the ISDA Auction Settlement Terms. In ISDA language, the auction has an **Initial Bidding Period** (Stage 1) and, if needed, a **Subsequent Bidding Period** (Stage 2). The inputs are:

- dealer **Initial Market Submissions** (bids/offers for deliverable obligations), and
- **Physical Settlement Requests** (who wants to end up buying/selling deliverables at the final price).

The output is a single **Auction Final Price** $FP_{100}$ that is used to compute cash settlement amounts.

### 40.3.1 Stage 1 — Initial Market Midpoint (IMM) and Open Interest (OI)

**Anchor (what ISDA defines):**

- From the set of valid Initial Market Submissions, the auction administrators compute an **Initial Market Midpoint (IMM)**. The procedure is rule-based: bids and offers are sorted, paired into “matched markets”, and the IMM is computed from the “best half” of the non-tradeable matched markets.
- The administrators then compute **Open Interest (OI)** by netting valid physical settlement buy and sell requests:

$$\boxed{\text{OI} = \sum \text{(Physical Settlement Buy Requests)} - \sum \text{(Physical Settlement Sell Requests)}}$$

**Expand (intuition):** IMM is a Stage-1 reference level summarizing dealer two-way markets. Open interest is the net amount of deliverables that the physical-settlement side of the market wants to acquire (positive) or deliver (negative) at the auction final price.

- $OI \gt 0$: net buy interest in deliverables.
- $OI \lt 0$: net sell interest in deliverables.

The administrators publish “Initial Bidding Information” including the direction/size of OI and the IMM. The rules also define **Adjustment Amounts** that can arise from “tradeable markets” (where a submitted bid touches/crosses a submitted offer).

**Check:** If $OI = 0$, the ISDA terms specify that no Stage-2 bidding occurs and the final price is set directly from the Stage-1 midpoint published by the administrators.

### 40.3.2 Stage 2 — Limit Orders, Cap Amounts, and the Auction Final Price

**Anchor (what ISDA defines):**

If $OI \ne 0$, the auction runs a Subsequent Bidding Period in which participants submit **limit orders on the opposite side of the market from the open interest**:

- If OI is a **bid to purchase** deliverables ($OI \gt 0$), Stage 2 consists of **offers**.
- If OI is an **offer to sell** deliverables ($OI \lt 0$), Stage 2 consists of **bids**.

The auction terms apply a **Cap Amount** around the IMM: extreme bids/offers beyond $IMM \pm \text{Cap Amount}$ are deemed at the capped level. After the Subsequent Bidding Period closes, administrators match the OI against the opposite-side limit orders starting from the best prices (lowest offer / highest bid) and moving outward until the OI is filled or there are no more opposite-side orders.

If the OI is filled, the **Auction Final Price** is set by the marginal (last) matched order price (subject to the Cap Amount rules). If the OI is *not* filled, the terms specify extreme outcomes (0% or 100%) for the Auction Final Price depending on direction.

**Expand (intuition):** Think of open interest as a single, auction-sized “market order” that must be cleared. Stage 2 supplies the depth of the order book on the opposite side. The cap amount keeps the final price from being determined by far-away prints relative to the Stage-1 midpoint.

**Check (toy clearing example):** Suppose Stage 1 yields $IMM=35.00$ and $OI=+150$ (a bid to purchase deliverables). In Stage 2, participants submit offers:

| Offer Price | Offer Volume |
|---|---:|
| 33.00 | 50 |
| 34.00 | 70 |
| 36.00 | 100 |

Administrators match the OI starting from the lowest offers:
- 33.00: fill 50 (remaining OI = 100)
- 34.00: fill 70 (remaining OI = 30)
- 36.00: fill 30 (OI filled; remaining offer volume is unfilled)

The marginal matched offer is 36.00, so (ignoring any cap adjustment in the settlement terms) **Auction Final Price = 36.00**.

**Check (why the final price can differ from IMM):** IMM is a Stage‑1 reference level, not the settlement price. The final price is set by where the Stage‑2 order book clears the open interest. In the toy example, clearing a net **buy** open interest against offers makes the marginal matched offer (36.00) the final price, even though the Stage‑1 midpoint was 35.00.

---

## 40.4 Historical Context and Examples

### 40.4.1 The Lehman Bankruptcy (September 2008)

The Lehman bankruptcy is a canonical illustration of why auctions matter. Widely cited estimates highlight a severe mismatch between the amount of CDS protection outstanding and the supply of deliverable Lehman obligations:

- **CDS outstanding:** approximately USD 400 billion
- **Debt outstanding:** approximately USD 155 billion
- **Ratio:** CDS notional was roughly 2.6× the available debt

Physical settlement for all contracts was mathematically impossible. Even if every dollar of Lehman debt were delivered multiple times (which makes no economic sense), there wouldn't be enough to settle all contracts.

The ISDA auction determined an Auction Final Price of 8.625 (per 100). This implied:
- Recovery rate: 8.625%
- Protection payout rate: 91.375%

In plain language: Lehman deliverables cleared at roughly eight cents on the dollar.

### 40.4.2 Historical Auction Results

If you need a historical Auction Final Price for a specific credit event, do not rely on “commonly cited” tables. ISDA publishes auction settlement terms and auction results (including the Auction Final Price) for each event on the CDS Determinations Committees website.

### 40.4.3 Why Cash Settlement Became Standard

Auction settlement grew out of a practical constraint: when CDS notional can exceed deliverable supply, “everyone physically settles” is not feasible. Market protocols introduced auction-based cash settlement as a fallback to avoid settlement-driven squeezes, and the March 2009 ISDA supplement then formalized (hardwired) determinations committees and auction settlement terms into the documentation for transactions that incorporate it.

On April 8, 2009, the market moved through a widely cited CDS standardization (“Big Bang”) with three main parts: auction hardwiring, standardizing trading conventions, and central clearing.

### 40.4.4 Earlier Precedents: The Delivery Option Problem

Even when physical settlement is feasible, it embeds a **delivery option**: the protection buyer can choose which eligible deliverable obligation to deliver. If different deliverables trade at different post-event prices (even within the same seniority), the choice matters. Auction settlement compresses the deliverable set into a single settlement price $FP_{100}$, but a hedge can still exhibit **basis risk** if the bond you care about is not close to the implied “CTD” value reflected in the auction price.

### 40.4.5 Index Constituent Defaults

When a constituent of a CDS index (CDX, iTraxx) experiences a credit event, the auction process applies to that name specifically. The auction-determined final price then flows through to index positions.

At a high level, a single-name credit event affects only the defaulted constituent’s slice of the index notional. If the index has $M$ constituents and total notional $N$, the affected notional is $N/M$.

Under cash settlement via auction, the index holder's economics are:

$$\text{Index Payout} = \frac{N}{M} \times \left(1 - \frac{FP_{100}}{100}\right)$$

where $N$ is the total index notional, $M$ is the number of index constituents, and $FP_{100}$ is the auction final price for the defaulted name.

> **Desk Reality:** Index default settlement is usually processed as a cash settlement on the affected slice plus a notional reduction.
> **Common break:** forgetting the $1/M$ scaling (or using the wrong $M$ for the specific index series/roll).
> **What to check:** compute affected notional $N/M$, apply the same auction payout formula, and reconcile the post-settlement index notional.

**Check (toy number):** Consider a 125-name index position with $N=USD 125\text{mm}$. One constituent defaults with $FP_{100}=40$. Affected notional is $125\text{mm}/125=USD 1\text{mm}$ and payout is $USD 1\text{mm}\times(1-0.40)=USD 0.6\text{mm}$. After settlement, the index notional reduces by $USD 1\text{mm}$. See Chapter 41 for full index mechanics.

---

## 40.5 The Cash Settlement Payout Formula

### 40.5.1 Notation

| Symbol | Definition |
|--------|------------|
| $N$ | CDS notional principal (USD) |
| $FP_{100}$ | Auction Final Price, points per 100 outstanding principal (e.g., 35 means 35% of outstanding principal) |
| $R$ | Recovery fraction: $R = FP_{100}/100$ |
| $s$ | CDS running spread (annualized, as a decimal) |
| $\alpha$ | Accrual fraction from last premium date to credit event date |
| $\text{OI}$ | Open interest (net physical settlement requests) |
| $\text{IMM}$ | Initial Market Midpoint from Stage 1 |

### 40.5.2 Core Payoff Identity

For a standard single-name CDS settled using the auction final price, the protection payout (cash received by the protection buyer) is:

$$\boxed{\text{Payout to Protection Buyer} = N \times \left(1 - \frac{FP_{100}}{100}\right)}$$

This is the same identity as $N(1-R)$ with $R := FP_{100}/100$.

**Check (quick number):** If $N=USD 100\text{mm}$ and $FP_{100}=35$, then payout $=100\text{mm}\times(1-0.35)=USD 65\text{mm}$.

### 40.5.3 Unit and Boundary Checks

**Unit check:**
- $N$: USD
- $FP_{100}/100$: dimensionless
- Payout: USD ✓

**Boundary conditions:**
- If $FP_{100} = 100$ (full recovery): Payout = 0
- If $FP_{100} = 0$ (total loss): Payout = $N$
- For $0 \le FP_{100} \le 100$: $0 \le \text{Payout} \le N$ ✓

**Sensitivity:** Each 1-point change in $FP_{100}$ changes payout by $0.01 \times N$.

### 40.5.4 Accrued Premium at Default

Premium payments stop after a credit event, but because premiums are paid in arrears, the protection buyer typically owes an accrued premium amount up to the event date.

For a contract with spread $s$ (annualized), notional $N$, and accrual fraction $\alpha$ since the last premium date:

$$\boxed{\text{Accrued Premium} = N \cdot s \cdot \alpha}$$

**Check (rule of thumb):** If the event is “roughly mid‑coupon” and the coupon period is roughly a quarter ($\Delta\approx 0.25$), then $\alpha\approx 0.125$ and accrued premium is on the order of $0.125\times N\times s$. Use this only as intuition; production accrual uses the actual event date and contract day count.

This is paid by the protection buyer to the seller and is separate from the protection payout.

> **Pitfall — CDS premium accrual on default:** forgetting the accrued premium cashflow (or getting its sign wrong).
> **Why it matters:** the net cash settlement is wrong even when the auction payout is correct.
> **Quick check:** compute $N\\,s\\,\alpha$ using the contract day count and confirm buyer net $=$ protection payout $ - N\\,s\\,\alpha$.

---

## 40.6 Governance: The Determinations Committee

**Anchor (what ISDA defines):** Under the March 2009 ISDA framework, Credit Derivatives Determinations Committees are established for purposes of making determinations for transactions that incorporate the March 2009 supplement, and ISDA serves as the committee secretary.

**Expand (why it matters):** On a desk, the DC is the governance layer that turns “news” into contract-relevant facts. DC determinations (and the related auction settlement terms) are what downstream systems use to drive settlement workflows and risk/P&L attribution for credit events.

**Check:** For any live settlement, treat the published DC determinations and auction terms as the authoritative inputs. Do not infer an event determination date, deliverable set, or auction schedule from headlines.

---

## 40.7 Worked Examples

### Example — Single-Name CDS Auction Cash Settlement (Including Accrued Premium)

**Context**
- You are long protection on a single name.
- A credit event occurs and the contract will cash settle using the published Auction Final Price.
- You want the *net* settlement cashflow: protection payout minus accrued premium.

**Timeline (Make Dates Concrete)**
- Trade date / accrual start: 2025-12-20 (illustration; aligns with standard quarterly CDS coupon dates)
- Next scheduled premium payment date (would have been): 2026-03-20
- Credit event date: 2026-02-10
- Auction Final Price Determination Date: 2026-02-18 (illustration; actual schedule is auction-specific)
- Cash settlement date: 2026-02-21 (illustration; actual schedule is auction-specific)

**Inputs**
- Notional: $N=USD 10{,}000{,}000$
- Running spread: $s=500\text{ bp}=0.05$ (annualized)
- Premium day count (toy assumption): ACT/360 simple accrual
- Days accrued since last premium date: 52 days $\Rightarrow \alpha=52/360$
- Auction Final Price: $FP_{100}=35.00$

**Outputs (What You Produce)**
- Protection payout (buyer receives): $N\left(1-\frac{FP_{100}}{100}\right)$
- Accrued premium (buyer pays): $N\\,s\\,\alpha$
- Net settlement (buyer receives): protection payout minus accrued premium
- Risk metric (explicit): final-price sensitivity to a +1 point bump in $FP_{100}$

**Step-by-step**
1. Recovery fraction: $R=FP_{100}/100=0.35$.
2. Protection payout: $10{,}000{,}000\times(1-0.35)=USD 6{,}500{,}000$.
3. Accrued premium: $10{,}000{,}000\times0.05\times(52/360)=USD 72{,}222.22$.
4. Net settlement to protection buyer: $USD 6{,}500{,}000-USD 72{,}222.22=USD 6{,}427{,}777.78$.

**Cashflows (positive = received by protection buyer)**
| Date | Cashflow | Explanation |
|---|---:|---|
| 2026-02-21 | +USD 6,500,000.00 | Protection payout from auction final price |
| 2026-02-21 | −USD 72,222.22 | Accrued premium owed up to the credit event |
| 2026-02-21 | **+USD 6,427,777.78** | Net settlement (what you should reconcile to) |

**P&L / Risk Interpretation**
- The auction final price is a direct P&L driver for credit-event settlement: for long protection, a higher $FP_{100}$ means a smaller payout.
- The accrued premium is usually smaller than the payout but is a common reconciliation break if omitted.

**Sanity Checks**
- Unit check: payout and accrued premium are in USD; $FP_{100}$ is “points per 100”; $\alpha$ is a year fraction.
- Sign check (long protection): $\Delta FP_{100}\gt 0 \Rightarrow \Delta \text{Payout}\lt 0$.
- Limit check: $FP_{100}=100 \Rightarrow$ payout $=0$; $FP_{100}=0 \Rightarrow$ payout $=N$.
- Accrued premium should be between 0 and the full‑period premium $N\\,s\\,\Delta(\text{last premium date},\text{next premium date})$. In this toy timeline the full coupon is about $USD 10\text{mm}\times 5\%\times 0.25 \approx USD 125\text{k}$, so $USD 72\text{k}$ is plausible.

**Debug Checklist (When Your Result Looks Wrong)**
- Did you use points (35) vs fractions (0.35) consistently?
- Is $\alpha$ computed with the contract day count and the correct start/end dates?
- Did you apply the correct direction (long vs short protection) when signing cashflows?

**References**
- See end-of-chapter References (ISDA auction terms for definitions; credit-derivatives texts for payoff and accrual conventions).

---

## 40.8 Risk and Measurement Considerations

### 40.8.1 Final-Price Sensitivity (Explicit Bump Definition)

Near a credit event, a simple and very “real” risk measure is the sensitivity of the protection payout to the auction final price.

- **Bump object:** $FP_{100}$ (Auction Final Price; points per 100 outstanding principal).
- **Bump size:** +1.00 point (e.g., 35.00 $\rightarrow$ 36.00).
- **Units:** currency per point (e.g., USD per 1 point) for the stated notional.
- **Sign convention:** from the perspective of **long protection** (protection buyer), the sensitivity is **negative**.

From $\text{Payout}=N\left(1-\frac{FP_{100}}{100}\right)$,

$$\boxed{\frac{\Delta \text{Payout}}{\Delta FP_{100}} = -\frac{N}{100}}$$

So a 10-point move in $FP_{100}$ changes payout by $0.10\times N$ (down in $FP_{100}$ is good for long protection; up is bad).

### 40.8.2 Deliverable Dispersion Risk

Even within the deliverable set, post-event prices can differ (e.g., because of accrued interest differences, maturity, or differing expectations about restructuring outcomes). This matters because the auction final price is a *single* number used for settlement, while a desk may care about the value of a *specific* bond.

**Implication for hedging:** If you hedge a specific bond with CDS, the hedge effectiveness depends on how that bond's post-event price compares to the CTD-referenced auction price. This is a source of basis risk—covered in detail in Chapters 43 and 44.

**Check (toy basis number):** If the auction final price is $35$ but the bond you care about trades at $25$, then cash settlement pays $0.65N$ while “deliver and receive par” economics on that bond would look like $0.75N$. The 10-point gap is a $0.10N$ basis difference.

### 40.8.3 Settlement Timing Risk

Physical settlement can extend for weeks (some sources cite up to 72 calendar days in certain setups). Auction-based cash settlement is more standardized, but the schedule is still auction-specific and published in the auction settlement terms. Timing risk shows up as uncertainty about:

- When the auction will occur
- What price will result
- When settlement funds will flow

---

## 40.9 Practical Notes

### 40.9.1 Day-of-Event Checklist

1. **Confirm reference entity and contracts impacted** — legal entity mapping matters
2. **Verify settlement type** — physical, cash, or auction-based fallback
3. **Record key data:** notional $N$, spread $s$, last premium date, event date (for accrual)
4. **Monitor DC determinations / auction terms:** Is an auction scheduled? What deliverables and schedule apply?
5. **Obtain auction final price** $FP_{100}$ when published
6. **Compute payout:** $N \times (1 - FP_{100}/100)$
7. **Reconcile signs:** buyer receives protection, pays accrued premium

> **Desk Reality:** Settlement is booked off a small set of fields: direction (long/short protection), notional $N$, contractual spread/coupon and premium day count, last premium date, credit event date, and the published $FP_{100}$.
> **Common break:** mixing “points per 100” with fractions (35 vs 0.35), or omitting accrued premium.
> **What to check:** recompute payout and $N\\,s\\,\alpha$ in a spreadsheet and confirm the net cash amount and sign.

### 40.9.2 Common Pitfalls

| Pitfall | Consequence |
|---------|-------------|
| Mixing price-per-100 with fraction (38 vs 0.38) | Factor of 100 error in payout |
| Forgetting accrued premium | Small but non-zero cash flow omitted |
| Assuming auction operational details | Verify against ISDA documentation |
| Treating auction price as "true recovery" for all obligations | Deliverable dispersion creates gaps |
| Ignoring restructuring clause | Affects deliverable range (see Chapter 39) |
| Applying single-name auction price incorrectly to index | Must account for $1/M$ weighting |

### 40.9.3 Verification Tests

1. **Payout bounds:** If $0 \le FP_{100} \le 100$, then $0 \le \text{Payout} \le N$
2. **Mirror symmetry:** Buyer's net = −(Seller's net)
3. **Physical/cash alignment:** Under stated assumptions, both settlement methods give same economics
4. **Index consistency:** Index payout = (1/M) × single-name payout formula

### 40.9.4 Information Sources

| Information Needed | Source |
|-------------------|--------|
| Auction final prices / auction terms | CDS Determinations Committees website (ISDA) and the published auction settlement terms for the event |
| DC determinations | CDS Determinations Committees website (ISDA) |
| Contract conventions | Your confirmation / definitions incorporated into the trade; CCP rulebooks if cleared |
| Deliverable obligations | Auction-specific settlement terms and DC publications |

---

## Summary

1. Auctions provide scalable cash settlement when CDS notional is large relative to deliverable supply (reducing settlement-driven short squeezes).
2. The auction output is a single **Auction Final Price** $FP_{100}$ (points per 100 outstanding principal) applied consistently across contracts on the same reference entity.
3. Protection payout maps directly from the quote: $N\left(1-\frac{FP_{100}}{100}\right)$; accrued premium $N\\,s\\,\alpha$ is separate and is a common reconciliation break.
4. In ISDA terms, the auction is two-stage: Stage 1 computes the **Initial Market Midpoint (IMM)** and **Open Interest (OI)**; Stage 2 matches opposite-side limit orders against the OI to set the Auction Final Price (with explicit cap/rounding rules).
5. Basis risk can remain: deliverable dispersion means the bond you care about may not track the auction-implied deliverable value.
6. For index defaults, the same auction process applies at the single-name level; the payout applies to the affected slice $N/M$ and index notional reduces accordingly.
7. For live events, use the published DC determinations and the auction settlement terms for the specific event rather than assuming timelines or conventions.

---

## Key Concepts

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| Auction Final Price $FP_{100}$ | Auction-determined settlement price (points per 100 outstanding principal) | Directly determines cash settlement payouts |
| Initial Market Midpoint (IMM) | Stage-1 midpoint computed from initial market submissions | Anchor level used in auction rules (including caps) |
| Open Interest (OI) | Net physical settlement buy requests minus sell requests | Determines which side submits limit orders in Stage 2 |
| Cap Amount | Capping rule around IMM applied to limit-order prices and/or final-price outcomes | Limits extreme prints from dominating the clearing price |
| Adjustment Amounts | Payments specified in the auction terms that can arise from “tradeable markets” in Stage 1 | Adds economic consequence to crossing/touching submissions |
| Determinations Committee (DC) | Governance process under the ISDA framework for making settlement-relevant determinations | Turns “events” into authoritative contract inputs |
| Accrued premium on default | $N\\,s\\,\alpha$ paid by the protection buyer up to the event date | Common operational break if omitted or mis-signed |
| Deliverable dispersion / basis | Your bond’s post-event price can differ from the auction-implied deliverable value | Hedge may not match the bond you care about |
| Index notional reduction | Index notional reduces by the affected slice $N/M$ after a constituent default | Prevents double-counting risk/cashflows post-default |

---

## Notation

| Symbol | Definition |
|--------|------------|
| $N$ | CDS notional principal (USD) |
| $FP_{100}$ | Auction Final Price, points per 100 outstanding principal |
| $R$ | Recovery fraction: $FP_{100}/100$ |
| $s$ | CDS spread (annualized) |
| $\alpha$ | Accrual fraction since last premium date |
| $\text{OI}$ | Open interest (net physical settlement requests) |
| $\text{IMM}$ | Initial Market Midpoint |
| $M$ | Number of index constituents |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What problem did the 2005 ISDA protocol address? | CDS notional exceeding supply of deliverable obligations |
| 2 | What does the CDS auction determine? | A single Auction Final Price $FP_{100}$ used for cash settlement |
| 3 | How does Hull describe what the auction targets? | "Mid-market value of the cheapest deliverable bond" |
| 4 | What is the cash settlement payout formula? | $N(1-R) = N(1 - FP_{100}/100)$ |
| 5 | How much Lehman CDS vs debt was outstanding in 2008? | ~$400B CDS vs ~$155B debt |
| 6 | What was Lehman's auction final price? | 8.625 per 100 (about 8.6 cents on the dollar) |
| 7 | What payout rate did Lehman protection buyers receive? | 91.375% of principal |
| 8 | Why was physical settlement impossible for Lehman? | CDS notional (2.6×) exceeded available deliverable debt |
| 9 | What is a "short squeeze" in this context? | Buyers competing for scarce deliverables push prices up |
| 10 | How do auctions prevent short squeezes? | Eliminate need to source physical deliverables |
| 11 | When are physical and cash settlement economically equivalent? | When auction final price equals CTD market price |
| 12 | If $FP_{100} = 35$, what is the recovery fraction $R$? | $R = 0.35$ |
| 13 | If $FP_{100}$ drops by 10 points, how does buyer payout change? | Increases by $0.10 \times N$ |
| 14 | What is "accrued premium at default"? | Premium accrued since last payment date, paid by buyer |
| 15 | Why is accrued premium separate from the protection payout? | It compensates the seller for protection already provided |
| 16 | What boundary check confirms payout formula correctness? | If $0 \le FP_{100} \le 100$, then $0 \le \text{Payout} \le N$ |
| 17 | If $FP_{100} = 100$, what is the protection payout? | Zero (full recovery, no loss) |
| 18 | If $FP_{100} = 0$, what is the protection payout? | $N$ (total loss, full notional paid) |
| 19 | Who organizes the CDS auction process? | ISDA |
| 20 | Where do you find the auction schedule and parameters (e.g., cap amount)? | In the event-specific auction settlement terms published for the auction |
| 21 | What cashflows stop after a credit event? | Regular premium payments cease |
| 22 | Does the same auction price apply to all contracts on a reference entity? | Yes — standardization across all contracts |
| 23 | If $N = USD 100\text{mm}$ and $FP_{100} = 35$, what is the payout? | $100\text{mm} \times 0.65 = USD 65\text{mm}$ |
| 24 | What documents specify auction operational details? | ISDA Auction Settlement protocols and Determinations Committee rules |
| 25 | Why might deliverables trade at different prices after default? | Accrued interest differences, differing restructuring expectations |
| 26 | What is "deliverable dispersion risk"? | Risk that specific hedged bond differs in price from CTD/auction value |
| 27 | What is the payout sensitivity per 1-point change in $FP_{100}$? | $0.01 \times N$ |
| 28 | For a $USD 50\text{mm}$ position, what payout change from $FP$ moving 35→40? | Decrease of $USD 2.5\text{mm}$ |
| 29 | What is the Cap Amount used for in Stage 2? | It caps extreme limit-order prices around the Stage-1 midpoint (IMM), per the auction terms |
| 30 | What should you verify on a real trade before computing settlement? | Settlement type, auction price, notional, accrual conventions |
| 31 | What is "open interest" in a CDS auction? | Net physical settlement requests (buy minus sell) |
| 32 | In Stage 2, which side of limit orders are accepted? | The side opposite the open interest: offers if $OI \gt 0$, bids if $OI \lt 0$ |
| 33 | What is the Initial Market Midpoint (IMM)? | Midpoint calculated from Stage 1 dealer quotes |
| 34 | How is the Auction Final Price set when open interest is filled? | By the marginal (last) matched limit-order price, subject to any cap rules in the settlement terms |
| 35 | What does the ISDA Determinations Committee do? | Makes settlement-relevant determinations under the ISDA framework (e.g., credit event determinations) and publishes them |

---

## Mini Problem Set

**Solution Sketches (Selected) are below.**

1. A CDS has $N = USD 12\text{mm}$ and $FP_{100} = 25$. Compute payout and implied recovery.

2. If $N = USD 40\text{mm}$, compare payouts for $FP_{100} = 15$ vs $FP_{100} = 55$. What is the difference?

3. A CDS has spread $s = 500$ bps and notional $N = USD 30\text{mm}$. A credit event occurs 1 month into the quarter. Using $\alpha = 1/12$, compute accrued premium.

4. A protection buyer has $N = USD 50\text{mm}$. If $FP_{100}$ is 10 points lower than expected, what is the payout surprise?

5. Show that $0 \le \text{payout} \le N$ when $0 \le FP_{100} \le 100$.

6. Given $N = USD 20\text{mm}$, compute payout for $R = 0.30$ and $R = 0.60$. Interpret.

7. Construct net cashflows for buyer and seller including $USD 0.05\text{mm}$ accrued premium and $USD 6.0\text{mm}$ protection payout.

8. If Lehman's auction produced 91.375% payout rate, what would be the payout on $USD 200\text{mm}$ notional?

9. Explain why a short squeeze can reduce protection payout under physical settlement.

10. Provide a scenario where cash-settled payoff might differ from your specific bond's post-default value.

11. Using Hull's statement that the auction determines "mid-market value of the cheapest deliverable bond," explain why this aligns with physical settlement economics.

12. A portfolio has three CDS positions: $N_1 = USD 10\text{mm}$, $N_2 = USD 25\text{mm}$, $N_3 = USD 15\text{mm}$. If all default with $FP_{100} = 40$, compute total payout.

13. Describe how final price uncertainty affects risk management near distress.

14. Why did O'Kane say the 2005 protocol was introduced? What specific problem did it solve?

15. List five data items needed to process a credit event settlement for a CDS position.

16. **(Simulation)** Suppose Stage 1 results are IMM = 40.00 and OI = +USD 100mm (a bid to purchase). Stage 2 therefore consists of *offers* (opposite side of OI). Given the following Stage 2 offers, find the Auction Final Price (ignore any cap adjustments):
    - 40.00: USD 20mm
    - 39.50: USD 40mm
    - 39.00: USD 50mm
    - 38.50: USD 30mm

17. A CDX.NA.IG position has $USD 62.5\text{mm}$ notional (125 names). One name defaults with $FP_{100} = 40$. Calculate the payout and remaining index notional.

18. **(Reasoning)** In an auction where Stage 2 accepts offers (OI > 0), explain why a participant with long protection might submit aggressive offers, and what constrains this strategy.

---

### Solution Sketches (Selected)

**1.** $R = 0.25$. Payout $= 12\text{mm} \times (1 - 0.25) = USD 9\text{mm}$.

**2.** $FP = 15 \Rightarrow 40\text{mm} \times 0.85 = USD 34\text{mm}$; $FP = 55 \Rightarrow 40\text{mm} \times 0.45 = USD 18\text{mm}$. Difference = $USD 16\text{mm}$.

**3.** Accrued premium $= 30\text{mm} \times 0.05 \times (1/12) = USD 0.125\text{mm}$.

**4.** Surprise $= N \times 0.10 = 50\text{mm} \times 0.10 = USD 5\text{mm}$.

**5.** Since $FP_{100}/100 \in [0,1]$, we have $1 - FP_{100}/100 \in [0,1]$. Multiplying by $N \ge 0$ preserves bounds.

**6.** $R = 0.30 \Rightarrow 20\text{mm} \times 0.70 = USD 14\text{mm}$; $R = 0.60 \Rightarrow 20\text{mm} \times 0.40 = USD 8\text{mm}$. Higher recovery → lower payout.

**7.** Buyer net $= +6.0 - 0.05 = +USD 5.95\text{mm}$; seller net $= -6.0 + 0.05 = -USD 5.95\text{mm}$.

**8.** Payout $= 200\text{mm} \times 0.91375 = USD 182.75\text{mm}$.

**9.** Under physical settlement, if CDS notional exceeds deliverable bonds, protection buyers must compete to source bonds. This buying pressure drives prices up (short squeeze), so the bond costs more to acquire. Since the protection buyer delivers the bond for par, a higher acquisition cost means lower net gain (effectively lower payout).

**10.** Suppose your bond has unusual features (long maturity, low coupon) that make it trade at 25 post-default, while the CTD trades at 35. The auction final price is 35 (CTD value), so cash settlement pays $N(1-0.35) = 0.65N$. But if you held the specific bond and physically settled, you could deliver the 25-price bond for par, gaining $N(1-0.25) = 0.75N$. The 10-point deliverable dispersion creates a $0.10N$ difference.

---

**16.** Since $OI \gt 0$, Stage 2 consists of offers. Sort offers from lowest to highest and accumulate volume until it reaches $100\text{mm}$:
- 38.50: $30\text{mm}$ (remaining OI $=70\text{mm}$)
- 39.00: $30+50=80\text{mm}$ (remaining OI $=20\text{mm}$)
- 39.50: $80+40=120\text{mm}$ (OI filled)

The marginal matched offer is 39.50, so **Auction Final Price = 39.50** (ignoring any cap adjustments).

**17.** Affected notional = $62.5\text{mm}/125 = USD 0.5\text{mm}$. Payout = $0.5\text{mm} \times (1-0.40) = USD 0.3\text{mm}$. Remaining notional = $62.5\text{mm} - 0.5\text{mm} = USD 62.0\text{mm}$.

**18.** Long protection benefits from a lower $FP_{100}$ (higher payout). If Stage 2 accepts offers, submitting low-priced offers can pull the marginal matched offer down and reduce the final price. Constraints include: (1) you may have to actually sell deliverables at the prices you submit; (2) limit orders are capped around IMM by the Cap Amount; and (3) Stage-1 information (IMM and OI) is published, so other participants can respond with their own orders.

---

## References

- ISDA (2009-03-17). *Form of Credit Derivatives Auction Settlement Terms* (Annex B, “Auction Hardwiring” documentation). Defines Initial Market Midpoint, Open Interest, Cap Amount, the matching algorithm, and “Auction Final Price”.  
  Available at: https://www.cdsdeterminationscommittees.org/companies/auctionhardwiring/docs/Auction-Settlement-Terms-CLEAN.doc
- ISDA (2009-03-17). *Credit Derivatives Determinations Committees Rules* (Annex A, “Auction Hardwiring” documentation). Defines DC structure and ISDA as DC Secretary.  
  Available at: https://www.cdsdeterminationscommittees.org/companies/auctionhardwiring/docs/Determinations-Committees-Rules.doc
- ISDA (published 2009-03-12). *2009 ISDA Credit Derivatives Determinations Committees and Auction Settlement Supplement to the 2003 ISDA Credit Derivatives Definitions*. Links determinations committees and auction settlement into the definitions for transactions that incorporate the supplement.  
  Available at: https://www.cdsdeterminationscommittees.org/companies/auctionhardwiring/docs/Supplement-CLEAN.doc
- Hull, John C. (2022). *Options, Futures, and Other Derivatives* (11th ed.). Pearson. Sections on credit default swaps and auction settlement (Lehman example; cash-settlement mapping).
- Hull, John C. (2015). *Risk Management and Financial Institutions* (4th ed.). Wiley. Credit derivatives / CDS market discussion (mentions the two-stage auction process).
- O’Kane, Dominic. *Modelling Single-name and Multi-name Credit Derivatives*. Wiley. CDS mechanics (premium/protection legs), accrued premium at default, and index constituent default handling.
- Kosowski, Robert L. & Neftci, Salih N. (3rd ed.). *Principles of Financial Engineering*. Section 18.4.1 on CDS standardization (“Big Bang”), auction hardwiring, and determinations committees.
- Privault, Nicolas (May 2024). *Notes on Financial Risk and Analytics*. Chapter 11 “Credit Derivatives” (CDS premium leg vs protection leg framing and notation).
