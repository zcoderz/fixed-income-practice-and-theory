# Chapter 40: The CDS Auction Process — What It Does and Why It Exists

---

## Introduction

When Lehman Brothers filed for bankruptcy in September 2008, the credit derivatives market faced an unprecedented challenge. Hull documents that "there was about $400 billion of CDS contracts and $155 billion of Lehman debt outstanding." If every protection buyer tried to source bonds for physical settlement, the resulting scramble would have been chaotic—and the prices paid would have little resemblance to fair value.

The solution was a cash settlement auction organized by ISDA. Within days, participating dealers submitted bids and offers, and the auction determined a "final price" of 8.625 cents on the dollar. This meant protection buyers received 91.375% of notional, and this single price applied uniformly to all contracts on Lehman. The auction transformed what could have been market chaos into an orderly settlement process.

This chapter explains what the CDS auction accomplishes and why it exists. We focus on the auction as the mechanism that makes cash settlement work at scale. Chapter 39 covered the underlying mechanics—credit events, physical versus cash settlement, the delivery option, and cheapest-to-deliver dynamics. Here we build on that foundation to understand:

1. **Why auctions became necessary** — the problem of notional exceeding deliverable supply
2. **What the auction determines** — the final price for cash settlement
3. **Settlement economics** — mapping auction prices to protection payouts
4. **Historical examples** — Lehman and the evolution of settlement practice
5. **What remains uncertain** — operational details not specified in our primary sources

We are explicit about what the reference books cover and what requires additional documentation (such as ISDA's auction protocols). Where operational details are not specified in our sources, we say "I'm not sure" rather than speculate.

---

## 40.1 The Problem: When CDS Notional Exceeds Debt Outstanding

### 40.1.1 Physical Settlement and Scarcity

Chapter 39 established that physical settlement requires the protection buyer to deliver face value of eligible obligations and receive par in cash. O'Kane describes the mechanics: "The protection buyer delivers face value of deliverable obligations to the protection seller. In return, the protection seller makes a simultaneous payment of the face value in cash to the protection buyer."

This works well when the amount of CDS protection outstanding is modest relative to the available debt. But the credit derivatives market grew rapidly through the 2000s, and a structural problem emerged: CDS notional outstanding could far exceed the supply of deliverable obligations.

Hull provides the stark example: when Lehman defaulted, there was "$400 billion of CDS contracts and $155 billion of Lehman debt outstanding." The implication is immediate: if every protection buyer needed to source deliverable bonds, the combined buying pressure would create a massive short squeeze—pushing bond prices up, reducing the effective protection payout, and potentially destabilizing markets.

### 40.1.2 The 2005 ISDA Protocol

O'Kane documents the market's solution: "In situations where the outstanding notional of derivative contracts exceeds the supply of deliverable obligations, a new protocol was introduced by ISDA in 2005 which allows a fallback to cash settlement in which case an auction method is used to determine a cash settlement price."

This was a pivotal development. The 2005 protocol established that when physical settlement was impractical due to supply constraints, contracts could settle via cash payment based on an auction-determined price. The auction would produce a single price representing the market value of deliverable obligations, and all contracts would settle against that price.

The economic logic is compelling:

1. **Avoids short squeezes:** Protection buyers don't need to source scarce deliverables, eliminating artificial buying pressure that would distort prices
2. **Standardization:** A single auction price applies to all contracts on the same reference entity, rather than forcing bilateral negotiations
3. **Operational simplicity:** Removes the logistics of sourcing, verifying, and delivering physical obligations

> **Analogy: The Market Clearing**
>
> Imagine 100 people promised to deliver a specific rare painting (the bond). There are only 5 paintings in existence.
>
> *   **Physical Settlement Madness**: 100 people fight over 5 paintings. The price skyrockets to \$100 million. The protection sellers lose everything.
> *   **The Auction Solution**: Everyone agrees to meet in a room. We ask the 5 owners: "What is a fair price to sell?" We ask the 100 buyers: "What is a fair price to settle?"
> *   **Result**: We agree on a single price (e.g., \$5 million). The 95 people who can't find a painting just pay the cash difference. Order is restored.

---

## 40.2 What the Auction Determines

### 40.2.1 The Final Price Concept

The auction produces a "final price" (or "recovery price") that determines cash settlement payoffs. Hull describes this as "the mid-market value of the cheapest deliverable bond several days after the credit event."

This description reveals two important features:

**Mid-market value:** The auction seeks to establish a fair market price, not a distressed fire-sale price or an artificially inflated price driven by settlement demand. The mid-market concept aims for the price at which willing buyers and sellers would transact in normal conditions.

**Cheapest deliverable:** The auction price relates to the cheapest-to-deliver (CTD) concept from Chapter 39. Under physical settlement, rational protection buyers would deliver the lowest-priced eligible obligation. The auction price should reflect this CTD value to maintain economic equivalence between settlement methods.

### 40.2.2 Connection to Settlement Economics

Once the auction establishes a final price, cash settlement follows directly. O'Kane describes the cash settlement payoff: "The protection seller pays the protection buyer the face value of the protection minus the recovery price of the reference obligation in cash."

Using our notation where $FP_{100}$ denotes the final price per 100 of face value:

$$\boxed{\text{Protection Payout} = N \times \left(1 - \frac{FP_{100}}{100}\right) = N(1-R)}$$

where $R = FP_{100}/100$ is the recovery fraction.

**Lehman example:** The auction determined $FP_{100} = 8.625$, implying $R = 0.08625$. For a protection buyer with $100 million notional:

$$\text{Payout} = \$100\text{mm} \times (1 - 0.08625) = \$91.375\text{mm}$$

This "cash payout to the buyers of protection... was 91.375% of principal" as Hull reports.

### 40.2.3 Why Physical and Cash Settlement Should Align

When does cash settlement deliver the same economic outcome as physical settlement? Chapter 39 derived that physical settlement payoff equals $N(1 - P_{CTD}/100)$, where $P_{CTD}$ is the cheapest deliverable price.

**Alignment condition:** If the auction final price equals the CTD market price—that is, if $FP_{100} = P_{CTD}$—then:

$$\Pi_{\text{cash}} = N\left(1 - \frac{FP_{100}}{100}\right) = N\left(1 - \frac{P_{CTD}}{100}\right) = \Pi_{\text{phys}}$$

This is precisely what the auction design aims to achieve. By targeting the "mid-market value of the cheapest deliverable bond," the auction internalizes CTD economics into a single settlement price, making cash settlement economically equivalent to physical settlement.

---

## 40.3 The Auction Mechanism

### 40.3.1 What the Sources Support

Hull describes that "an ISDA-organized auction process is used to determine the mid-market value of the cheapest deliverable bond several days after the credit event." O'Kane confirms that "an auction method is used to determine a cash settlement price."

**What we know with certainty from the sources:**
- ISDA organizes the auction process
- The auction occurs several days after the credit event
- The auction determines a cash settlement price
- The price is described as the "mid-market value of the cheapest deliverable bond"
- The same final price applies to all contracts on the same reference entity
- Hull mentions a "two-stage" process

> **Deep Dive: The Two-Stage Auction**
>
> How does ISDA calculate the price?
>
> **Stage 1: The Initial Market Midpoint (IMM)**
> *   Dealers submit two-way quotes (e.g., Bid 30, Offer 32).
> *   We act on these quotes to stop dealers from lying. If you quote too high, you have to buy!
> *   This sets the "Fair Value" range.
>
> **Stage 2: Limit Orders**
> *   Real investors (and banks) submit orders to buy or sell the actual bonds at specific prices.
> *   "I will buy $50mm bonds if the price is 28."
> *   The auction matches the net buying interest against the net selling interest to find the **Final Price**.

### 40.3.2 What Is Not Specified in the Sources

I'm not sure about the full operational details of the auction mechanism from the primary sources. The following aspects are not specified in the excerpts I have verified:

- How dealers submit bids and offers and what constraints apply
- How "open interest" (net physical settlement demand/supply) is calculated and incorporated
- The precise algorithm for converting submissions into the final price
- Exact timelines (cutoffs, publication times, settlement date conventions)
- The role and composition of the ISDA Determinations Committee

**External documentation needed:** The ISDA Credit Derivatives Auction Settlement documentation, ISDA Determinations Committee rules, and the applicable ISDA Credit Derivatives Definitions contain the operational details. Our primary sources (Hull, O'Kane) point readers to these documents for full specifications rather than reproducing them.

---

## 40.4 Historical Context and Examples

### 40.4.1 The Lehman Bankruptcy (September 2008)

The Lehman default provides the canonical illustration of why auctions matter. Hull documents the scale mismatch:

- **CDS outstanding:** approximately $400 billion
- **Debt outstanding:** approximately $155 billion
- **Ratio:** CDS notional was roughly 2.6× the available debt

Physical settlement for all contracts was mathematically impossible. Even if every dollar of Lehman debt were delivered multiple times (which makes no economic sense), there wouldn't be enough to settle all contracts.

The ISDA auction determined a final price of 8.625 per 100 of face value. This implied:
- Recovery rate: 8.625%
- Protection payout rate: 91.375%

Hull notes this was approximately "eight cents on the dollar" in recovery—reflecting severe impairment in Lehman's credit quality.

### 40.4.2 Why Cash Settlement Became Standard

O'Kane explains the evolution: "The current market standard for default swaps is to prefer physical settlement over cash settlement. However... there is a trend towards cash settlement for CDS, especially in the CDS index and STCDO markets."

The 2005 protocol and subsequent refinements established auctions as the standard fallback mechanism. Hull notes that cash settlement with auctions is "now usual"—a significant shift from earlier practice where physical settlement was the norm.

### 40.4.3 Earlier Precedents: The Delivery Option Problem

Before auctions became standard, physical settlement exposed the market to delivery option dynamics. Chapter 39 covered O'Kane's Conseco example from September 2000, where banks exploited the delivery option by delivering long-maturity, deep-discount bonds (trading at 65-80) for par while shorter-dated loans traded higher.

Such exploitation illustrated the challenges of physical settlement when deliverables trade at dispersed prices. Auctions help address this by establishing a single settlement price that reflects CTD economics without requiring actual delivery.

---

## 40.5 The Cash Settlement Payout Formula

### 40.5.1 Notation

| Symbol | Definition |
|--------|------------|
| $N$ | CDS notional principal (USD) |
| $FP_{100}$ | Auction "final price" per 100 face (e.g., 35 means 35% of par) |
| $R$ | Recovery fraction: $R = FP_{100}/100$ |
| $s$ | CDS running spread (annualized, as a decimal) |
| $\alpha$ | Accrual fraction from last premium date to credit event date |

### 40.5.2 Core Payoff Identity

Hull states that "the payoff from a CDS is $L(1-R)$, where $L$ is the notional principal and $R$ is the recovery rate."

Using the auction final price:

$$\boxed{\text{Payout to Protection Buyer} = N \times \left(1 - \frac{FP_{100}}{100}\right)}$$

**Hull's worked example:** "Suppose the auction indicates that the bond is worth $35 per $100 of face value. The cash payoff would be $65 million" on $100 million notional.

Verification: $\$100\text{mm} \times (1 - 0.35) = \$65\text{mm}$ ✓

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

Hull emphasizes that "the regular payments from the buyer of protection to the seller of protection cease when there is a credit event. However, because these payments are made in arrears, a final accrual payment by the buyer is usually required."

For a contract with spread $s$ (annualized), notional $N$, and accrual fraction $\alpha$ since the last premium date:

$$\boxed{\text{Accrued Premium} = N \cdot s \cdot \alpha}$$

This is paid by the protection buyer to the seller and is separate from the protection payout.

---

## 40.6 Worked Examples

### Example A — Basic Cash Settlement Payout

**Given:** $N = \$10\text{mm}$, $FP_{100} = 38$

**Step 1:** Recovery fraction
$$R = \frac{38}{100} = 0.38$$

**Step 2:** Payout to protection buyer
$$\text{Payout} = 10\text{mm} \times (1 - 0.38) = 10\text{mm} \times 0.62 = \$6.2\text{mm}$$

**Answer:** Payout = **$6.2 million**.

---

### Example B — Including Accrued Premium

**Given (following Hull's example structure):**
- $N = \$100\text{mm}$
- Running spread $s = 90$ bps $= 0.0090$
- Credit event occurs 2 months into accrual period ($\alpha = 2/12$)
- Final price $FP_{100} = 35 \Rightarrow R = 0.35$

**Step 1:** Protection payout
$$\text{Payout} = 100\text{mm} \times (1 - 0.35) = \$65\text{mm}$$

**Step 2:** Accrued premium
$$\text{AccPrem} = 100\text{mm} \times 0.0090 \times \frac{2}{12} = \$0.15\text{mm}$$

**Step 3:** Net cashflows

| Party | Protection Leg | Accrued Premium | Net |
|-------|----------------|-----------------|-----|
| Buyer | +$65.00\text{mm}$ | −$0.15\text{mm}$ | +$64.85\text{mm}$ |
| Seller | −$65.00\text{mm}$ | +$0.15\text{mm}$ | −$64.85\text{mm}$ |

---

### Example C — Lehman Brothers Auction (Historical)

**Given (Hull):**
- CDS outstanding: ~$400 billion
- Debt outstanding: ~$155 billion
- Auction payout rate: 91.375% of principal

**Implied final price:**
$$FP_{100} = 100 - 91.375 = 8.625$$

**For a protection buyer with $50\text{mm}$ notional:**
$$\text{Payout} = 50\text{mm} \times 0.91375 = \$45.6875\text{mm}$$

The buyer receives ~$45.7 million, reflecting that Lehman bonds were valued at roughly 8.6 cents on the dollar.

---

### Example D — Sensitivity to Final Price

**Given:** $N = \$25\text{mm}$

| $FP_{100}$ | $R$ | Payout |
|------------|-----|--------|
| 35 | 0.35 | $25\text{mm} \times 0.65 = \$16.25\text{mm}$ |
| 40 | 0.40 | $25\text{mm} \times 0.60 = \$15.00\text{mm}$ |
| 45 | 0.45 | $25\text{mm} \times 0.55 = \$13.75\text{mm}$ |

**Interpretation:** Each 1-point increase in $FP_{100}$ reduces buyer payout by $0.25\text{mm}$ (= $25\text{mm} \times 0.01$).

---

### Example E — Physical vs Cash Settlement Equivalence

**Given:** $N = \$5\text{mm}$, CTD trades at $FP_{100} = 35$

**Physical settlement:**
- Buyer sources bonds with market value: $5\text{mm} \times 0.35 = \$1.75\text{mm}$
- Buyer delivers for par: $5.00\text{mm}$
- Economic gain: $5.00\text{mm} - 1.75\text{mm} = \$3.25\text{mm}$

**Cash settlement:**
$$\text{Payout} = 5\text{mm} \times (1 - 0.35) = \$3.25\text{mm}$$

**Result:** Under the assumption that $FP_{100}$ equals the CTD market price, both settlement methods deliver identical economics.

---

### Example F — Recovery Shift P&L Impact

**Given:** $N = \$20\text{mm}$, expected recovery $R^{\text{exp}} = 0.40$, realized auction recovery $R^{\text{real}} = 0.25$

**Expected payout:**
$$20\text{mm} \times (1 - 0.40) = \$12.0\text{mm}$$

**Realized payout:**
$$20\text{mm} \times (1 - 0.25) = \$15.0\text{mm}$$

**Auction surprise (buyer's gain):**
$$15.0\text{mm} - 12.0\text{mm} = \$3.0\text{mm}$$

Lower-than-expected recovery means higher-than-expected payout for protection buyers.

---

### Example G — Portfolio Settlement

**Given three CDS positions:**

| Trade | Notional | $FP_{100}$ | $R$ | Payout |
|-------|----------|------------|-----|--------|
| 1 | $10\text{mm}$ | 30 | 0.30 | $7.0\text{mm}$ |
| 2 | $15\text{mm}$ | 55 | 0.55 | $6.75\text{mm}$ |
| 3 | $5\text{mm}$ | 80 | 0.80 | $1.0\text{mm}$ |

**Total payout:** $7.0 + 6.75 + 1.0 = \$14.75\text{mm}$

**Concentration:** Trade 1 contributes $7.0/14.75 \approx 47\%$ of total payout despite having the smallest notional—because it has the lowest recovery.

---

### Example H — Deliverable Dispersion and Final Price

**Scenario:** Two deliverables trade at prices 32 and 48 after a credit event.

**If auction final price = 32 (CTD value):**
$$\text{Payout per }\$10\text{mm} = 10\text{mm} \times (1 - 0.32) = \$6.8\text{mm}$$

**If auction final price = 40 (average):**
$$\text{Payout per }\$10\text{mm} = 10\text{mm} \times (1 - 0.40) = \$6.0\text{mm}$$

**Difference:** $0.8\text{mm}

The final price choice matters. Hull describes the auction as determining the "mid-market value of the cheapest deliverable bond," suggesting alignment with CTD economics.

---

### Example I — Complete Settlement Timeline Table

**Given:**
- $N = \$50\text{mm}$
- $s = 200$ bps
- Default occurs halfway through quarter: $\alpha = 0.125$
- $FP_{100} = 40 \Rightarrow R = 0.40$

**Amounts:**
- Protection payout: $50\text{mm} \times 0.60 = \$30.0\text{mm}$
- Accrued premium: $50\text{mm} \times 0.0200 \times 0.125 = \$0.125\text{mm}$

**Cashflow table (positive = received):**

| Item | Buyer | Seller |
|------|-------|--------|
| Protection leg | +$30.000\text{mm}$ | −$30.000\text{mm}$ |
| Accrued premium | −$0.125\text{mm}$ | +$0.125\text{mm}$ |
| **Net** | **+$29.875\text{mm}$** | **−$29.875\text{mm}$** |

---

### Example J — Why Auctions Avoid Short Squeezes

**Scenario:** Reference entity defaults with:
- CDS outstanding: $200 billion
- Deliverable debt: $50 billion

Under physical settlement, protection buyers controlling $200 billion notional would compete to acquire $50 billion of deliverable bonds. This 4:1 ratio would create extreme buying pressure, potentially driving bond prices from (say) 30 to 60—cutting protection payouts roughly in half.

Under auction-based cash settlement, there is no need to acquire physical bonds. The auction determines a price reflecting fair value without settlement-driven demand distortion.

**Economic impact:** If fair value is 30 but short-squeeze prices reach 60:
- Physical settlement payout: $N \times (1 - 0.60) = 0.40N$
- Cash settlement payout: $N \times (1 - 0.30) = 0.70N$

Protection buyers receive 75% more through auction-based settlement.

---

## 40.7 Risk and Measurement Considerations

### 40.7.1 Final Price Uncertainty

**What it is:** Risk that the realized auction price $FP_{100}$ differs from pre-event expectations.

**Why it matters:** For a protection buyer, lower-than-expected $FP_{100}$ means higher payout (favorable). For a seller, lower $FP_{100}$ means larger loss.

**Magnitude:** Each 10-point move in $FP_{100}$ changes payout by $0.10 \times N$.

**When it matters most:** Near distress, when the reference entity's debt is trading at uncertain levels and market participants disagree about recovery prospects.

### 40.7.2 Deliverable Dispersion Risk

Hull notes in a footnote that same-seniority bonds "may not sell for the same percentage of face value immediately after a default" due to "accrued interest differences and differing expectations about outcomes across bondholders."

**Implication for hedging:** If you hedge a specific bond with CDS, the hedge effectiveness depends on how that bond's post-event price compares to the CTD-referenced auction price. This is a source of basis risk—covered in detail in Chapters 43 and 44.

### 40.7.3 Settlement Timing Risk

O'Kane documents that physical settlement "can extend up to 72 calendar days after initial notification until a payment on the protection leg must be made." Cash settlement via auction compresses this timeline but still involves uncertainty about:

- When the auction will occur
- What price will result
- When settlement funds will flow

---

## 40.8 Practical Notes

### 40.8.1 Day-of-Event Checklist

1. **Confirm reference entity and contracts impacted** — legal entity mapping matters
2. **Verify settlement type** — physical, cash, or auction-based fallback
3. **Record key data:** notional $N$, spread $s$, last premium date, event date (for accrual)
4. **Obtain auction final price** $FP_{100}$ when published
5. **Compute payout:** $N \times (1 - FP_{100}/100)$
6. **Reconcile signs:** buyer receives protection, pays accrued premium

### 40.8.2 Common Pitfalls

| Pitfall | Consequence |
|---------|-------------|
| Mixing price-per-100 with fraction (38 vs 0.38) | Factor of 100 error in payout |
| Forgetting accrued premium | Small but non-zero cash flow omitted |
| Assuming auction operational details | Verify against ISDA documentation |
| Treating auction price as "true recovery" for all obligations | Deliverable dispersion creates gaps |
| Ignoring restructuring clause | Affects deliverable range (see Chapter 39) |

### 40.8.3 Verification Tests

1. **Payout bounds:** If $0 \le FP_{100} \le 100$, then $0 \le \text{Payout} \le N$
2. **Mirror symmetry:** Buyer's net = −(Seller's net)
3. **Physical/cash alignment:** Under stated assumptions, both settlement methods give same economics

---

## Summary

1. When CDS notional exceeds deliverable debt supply, **physical settlement becomes impractical** — Lehman's $400B CDS vs $155B debt illustrated this starkly

2. The **2005 ISDA protocol** introduced auction-based cash settlement as a fallback mechanism when physical settlement demand exceeds supply

3. The **auction determines a final price** representing the mid-market value of the cheapest deliverable bond several days after the credit event

4. The **cash settlement payout** is $N(1-R) = N(1 - FP_{100}/100)$, where $FP_{100}$ is the auction final price

5. **Auctions avoid short squeezes** by eliminating the need to source physical deliverables, preventing settlement-driven price distortion

6. The **same final price applies to all contracts** on the reference entity, providing standardization across thousands of bilateral positions

7. **Accrued premium at default** is paid by the buyer to the seller, separate from the protection payout

8. **Physical and cash settlement are economically equivalent** when the auction price equals the CTD market price

9. **Operational auction details** (bidding rules, timelines, algorithms) require ISDA documentation beyond what the primary sources specify

10. The auction mechanism transformed credit event settlement from potential chaos into an **orderly, standardized process**

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| Final price $FP_{100}$ | Auction-determined settlement price per 100 | Directly determines cash settlement payout |
| 2005 ISDA protocol | Introduced auction fallback for cash settlement | Resolved supply/demand mismatch when CDS > debt |
| Short squeeze avoidance | Eliminating physical sourcing pressure | Preserves fair-value settlement economics |
| CTD alignment | Auction targets CTD value | Makes cash and physical settlement equivalent |
| Accrued premium at default | Premium owed from last payment to event date | Ensures seller receives compensation for protection provided |

---

## Notation for This Chapter

| Symbol | Definition |
|--------|------------|
| $N$ | CDS notional principal (USD) |
| $FP_{100}$ | Final price per 100 of face |
| $R$ | Recovery fraction: $FP_{100}/100$ |
| $s$ | CDS spread (annualized) |
| $\alpha$ | Accrual fraction since last premium date |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What problem did the 2005 ISDA protocol address? | CDS notional exceeding supply of deliverable obligations |
| 2 | What does the CDS auction determine? | The final price (recovery price) for cash settlement |
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
| 20 | How many days after the credit event does the auction typically occur? | Several days (exact timing per ISDA documentation) |
| 21 | What cashflows stop after a credit event? | Regular premium payments cease |
| 22 | Does the same auction price apply to all contracts on a reference entity? | Yes — standardization across all contracts |
| 23 | If $N = \$100\text{mm}$ and $FP_{100} = 35$, what is the payout? | $100\text{mm} \times 0.65 = \$65\text{mm}$ |
| 24 | What documents specify auction operational details? | ISDA Auction Settlement protocols and Determinations Committee rules |
| 25 | Why might deliverables trade at different prices after default? | Accrued interest differences, differing restructuring expectations |
| 26 | What is "deliverable dispersion risk"? | Risk that specific hedged bond differs in price from CTD/auction value |
| 27 | What is the payout sensitivity per 1-point change in $FP_{100}$? | $0.01 \times N$ |
| 28 | For a $\$50\text{mm}$ position, what payout change from $FP$ moving 35→40? | Decrease of $\$2.5\text{mm}$ |
| 29 | What makes cash settlement "now usual" according to Hull? | ISDA-organized auction process standardization |
| 30 | What should you verify on a real trade before computing settlement? | Settlement type, auction price, notional, accrual conventions |

---

## Mini Problem Set

**Questions 1-8 have solution sketches below.**

1. A CDS has $N = \$12\text{mm}$ and $FP_{100} = 25$. Compute payout and implied recovery.

2. If $N = \$40\text{mm}$, compare payouts for $FP_{100} = 15$ vs $FP_{100} = 55$. What is the difference?

3. A CDS has spread $s = 500$ bps and notional $N = \$30\text{mm}$. A credit event occurs 1 month into the quarter. Using $\alpha = 1/12$, compute accrued premium.

4. A protection buyer has $N = \$50\text{mm}$. If $FP_{100}$ is 10 points lower than expected, what is the payout surprise?

5. Show that $0 \le \text{payout} \le N$ when $0 \le FP_{100} \le 100$.

6. Given $N = \$20\text{mm}$, compute payout for $R = 0.30$ and $R = 0.60$. Interpret.

7. Construct net cashflows for buyer and seller including $\$0.05\text{mm}$ accrued premium and $\$6.0\text{mm}$ protection payout.

8. If Lehman's auction produced 91.375% payout rate, what would be the payout on $\$200\text{mm}$ notional?

9. Explain why a short squeeze can reduce protection payout under physical settlement.

10. Provide a scenario where cash-settled payoff might differ from your specific bond's post-default value.

11. Using Hull's statement that the auction determines "mid-market value of the cheapest deliverable bond," explain why this aligns with physical settlement economics.

12. A portfolio has three CDS positions: $N_1 = \$10\text{mm}$, $N_2 = \$25\text{mm}$, $N_3 = \$15\text{mm}$. If all default with $FP_{100} = 40$, compute total payout.

13. Describe how final price uncertainty affects risk management near distress.

14. Why did O'Kane say the 2005 protocol was introduced? What specific problem did it solve?

15. List five data items needed to process a credit event settlement for a CDS position.

---

### Solution Sketches (Questions 1-8)

**1.** $R = 0.25$. Payout $= 12\text{mm} \times (1 - 0.25) = \$9\text{mm}$.

**2.** $FP = 15 \Rightarrow 40\text{mm} \times 0.85 = \$34\text{mm}$; $FP = 55 \Rightarrow 40\text{mm} \times 0.45 = \$18\text{mm}$. Difference = $\$16\text{mm}$.

**3.** Accrued premium $= 30\text{mm} \times 0.05 \times (1/12) = \$0.125\text{mm}$.

**4.** Surprise $= N \times 0.10 = 50\text{mm} \times 0.10 = \$5\text{mm}$.

**5.** Since $FP_{100}/100 \in [0,1]$, we have $1 - FP_{100}/100 \in [0,1]$. Multiplying by $N \ge 0$ preserves bounds.

**6.** $R = 0.30 \Rightarrow 20\text{mm} \times 0.70 = \$14\text{mm}$; $R = 0.60 \Rightarrow 20\text{mm} \times 0.40 = \$8\text{mm}$. Higher recovery → lower payout.

**7.** Buyer net $= +6.0 - 0.05 = +\$5.95\text{mm}$; seller net $= -6.0 + 0.05 = -\$5.95\text{mm}$.

**8.** Payout $= 200\text{mm} \times 0.91375 = \$182.75\text{mm}$.

---

## Source Map

### (A) Verified Facts (Source-Backed)

| Fact | Source |
|------|--------|
| Lehman: $400B CDS, $155B debt outstanding; 91.375% payout | Hull Ch 25 |
| "Not uncommon for the volume of CDSs on a company to be greater than its debt" | Hull Ch 25 |
| "As is now usual, an ISDA-organized auction process is used to determine the mid-market value of the cheapest deliverable bond several days after the credit event" | Hull Ch 25 |
| "In situations where the outstanding notional of derivative contracts exceeds the supply of deliverable obligations, a new protocol was introduced by ISDA in 2005 which allows a fallback to cash settlement in which case an auction method is used to determine a cash settlement price" | O'Kane Ch 10 |
| "The payoff from a CDS is $L(1-R)$, where $L$ is the notional principal and $R$ is the recovery rate" | Hull Ch 25 |
| "The regular payments from the buyer of protection to the seller of protection cease when there is a credit event. However, because these payments are made in arrears, a final accrual payment by the buyer is usually required" | Hull Ch 25 |
| Cash settlement: "The protection seller pays the protection buyer the face value of the protection minus the recovery price of the reference obligation in cash. The recovery price is determined by a dealer poll or auction process" | O'Kane Ch 5 |
| Physical settlement: "The protection buyer delivers face value of deliverable obligations to the protection seller. In return, the protection seller makes a simultaneous payment of the face value in cash to the protection buyer" | O'Kane Ch 5 |
| Same-seniority bonds "may not sell for the same percentage of face value immediately after a default" due to "accrued interest differences and differing expectations" | Hull Ch 25 (footnote) |
| Physical settlement timeline can extend "up to 72 calendar days after initial notification" | O'Kane Ch 5 |

### (B) Reasoned Inference (Derived from A)

| Inference | Derivation |
|-----------|------------|
| Cash settlement payout formula $N(1-R) = N(1-FP_{100}/100)$ | Direct from Hull's payoff statement |
| Physical/cash equivalence when $FP_{100} = P_{CTD}$ | Algebraic comparison of settlement payoffs |
| Payout sensitivity = $0.01 \times N$ per point | Differentiation of payout formula |
| Short squeeze risk reduces effective protection | CDS > debt supply creates buying pressure; reasoned from scarcity economics |
| Auctions eliminate short squeeze by removing need to source deliverables | Logical implication of cash settlement mechanics |

### (C) Flagged Uncertainties

- **Full auction operational mechanics:** I'm not sure about the two-stage process details, bid/offer constraints, open interest calculation, timeline milestones, or Determinations Committee procedures. Hull mentions "two-stage" but does not detail the mechanics. To be certain, we would need the ISDA Credit Derivatives Auction Settlement documentation.

- **Exact timing conventions:** I'm not sure about the precise number of days between credit event and auction, publication times, or settlement dates without the specific auction protocol.

- **How final price relates to CTD across all protocols:** I'm not sure whether every auction protocol targets CTD specifically or uses different pricing approaches. Hull's description of "mid-market value of the cheapest deliverable bond" is suggestive but not a complete protocol specification.

- **Determinations Committee procedures:** I'm not sure about modern DC rules for declaring credit events or determining settlement terms without the relevant ISDA governance documents.
