# Chapter 39: CDS Credit Events and Settlement

---

## Introduction

When a company defaults, a CDS contract must answer two questions: *Did the contract trigger?* and *How much does the protection seller owe?* The first question is about **credit events**—the legal triggers that activate protection. The second is about **settlement**—the mechanics that convert a triggering event into actual cash flows.

These questions seem straightforward until you consider the details. A "default" on a bond is not the same as a "credit event" in a CDS contract. A restructuring that changes loan terms might trigger some CDS contracts but not others, depending on their documentation. And even after a credit event, the protection payment depends critically on which bonds the buyer can deliver and at what price.

This chapter examines the mechanics that determine what you actually receive when protection is triggered. We cover:

1. **Credit events** — The contractual triggers that activate the protection leg, and why they differ from bond default definitions
2. **Hard vs soft credit events** — Why restructuring is different from bankruptcy, and why that difference matters for settlement
3. **Physical vs cash settlement** — Two ways to effect the protection payment, with different risk profiles
4. **The delivery option and cheapest-to-deliver** — Why the buyer's choice of what to deliver creates embedded optionality
5. **Restructuring clauses** — How the market evolved to limit delivery option value

Understanding these mechanics is essential because they determine the actual P&L when protection is triggered. A protection buyer who doesn't understand the delivery option may leave money on the table. A protection seller who ignores deliverable scarcity may face larger losses than expected.

---

## 39.1 Credit Events: The Contractual Trigger

### 39.1.1 Credit Event vs Default

O'Kane provides the key distinction: "The credit event is the legal term for the event which triggers the payment of the protection leg. It is an event which is similar to default but, as we will see, it is not exactly the same as the event of default as defined by the rating agencies."

This distinction matters because CDS contracts use a **contractual** definition of default, specified in ISDA documentation, rather than an **economic** notion of distress. A company can be in severe financial trouble without triggering a credit event, and conversely, certain technical triggers may activate CDS protection even when the company continues operating.

**Practical implication:** When analyzing CDS exposure, you must reference the specific contract documentation—not general notions of "default." The term "default" is used loosely in market language when the precise term should be "credit event."

### 39.1.2 The Six Standard Credit Events

O'Kane's Table 5.1 classifies the standard credit events used in CDS contracts. The market divides these into **hard** and **soft** events based on their economic implications.

| Credit Event | Type | Description |
|--------------|------|-------------|
| **Bankruptcy** | Hard | Corporate becomes insolvent or is unable to pay its debts. Not relevant for sovereign issuers. |
| **Failure to pay** | Hard | Failure to make due payments, taking into account a grace period. |
| **Obligation acceleration** | Hard | Obligations become due and payable early due to default or similar trigger. Used mostly in emerging market contracts. |
| **Obligation default** | Hard | Obligations have become due prior to maturity. Rarely used. |
| **Repudiation/moratorium** | Hard | Reference entity or government rejects or challenges validity of obligations. Used in emerging market sovereign CDS. |
| **Restructuring** | Soft | Changes in debt obligations associated with credit deterioration, excluding favorable renegotiations. |

O'Kane emphasizes that "restructuring is the only soft credit event." This classification has profound implications for settlement, which we explore in Section 39.4.

### 39.1.3 Why Hard Events Make Debt Trade at One Price

O'Kane explains the economic logic behind the hard/soft distinction: hard credit events "would cause all the debt of the reference entity to become immediately due and payable and hence trade at the same price. It is what the rating agencies would traditionally call default."

When bankruptcy occurs, the capital structure flattens—all senior unsecured debt has the same claim on assets, regardless of original maturity or coupon. A 2-year bond and a 30-year bond both trade at approximately the same recovery price because both represent equivalent claims in bankruptcy.

### 39.1.4 Why Restructuring Is Different

O'Kane continues: "Following a restructuring event, debt can continue trading with a term structure of prices. Short dated assets will tend to trade at higher prices than long dated assets, and assets with higher coupons will trade with a higher price than those with lower coupons."

After a restructuring, the company survives with modified debt terms. The term structure of bond prices remains intact—shorter maturities carry less risk than longer ones. This price dispersion across deliverable obligations is what makes restructuring special for CDS settlement, because the protection buyer can choose which obligation to deliver.

> **Decision Tree: Is it a Credit Event?**
>
> 1.  **Did they fail to pay?** (>$1M, after Grace Period) $\to$ **YES**.
> 2.  **Did they file for Bankruptcy?** $\to$ **YES**.
> 3.  **Did they Restructure?**
>     *   Did they change the coupon/maturity? $\to$ YES.
>     *   Was it forced (distressed)? $\to$ YES.
>     *   Is the agreement binding on all holders? $\to$ YES.
>     *   (If all YES) $\to$ **Review Restructuring Clause (Mod-Re / No-Re)**.

---

## 39.2 Physical Settlement

### 39.2.1 Mechanics

O'Kane describes physical settlement precisely: "The protection buyer delivers face value of deliverable obligations to the protection seller. In return, the protection seller makes a simultaneous payment of the face value in cash to the protection buyer."

The economics are straightforward. If the buyer delivers $N$ face value of obligations and receives $N$ in cash, the buyer's economic gain equals $N$ minus the market value of what was delivered. For an obligation trading at price $P_i$ (per 100 of face):

$$\boxed{\Pi_{\text{phys}}(i) = N\left(1 - \frac{P_i}{100}\right)}$$

If the obligation trades at 35 cents on the dollar, the buyer delivers something worth $0.35N$ and receives $N$—a net gain of $0.65N$.

### 39.2.2 The Settlement Timeline

O'Kane describes the physical settlement process following a credit event:

**Step 1 — Credit Event Notice:** The protection buyer must notify the protection seller that a credit event has occurred. O'Kane specifies: "The event must be evidenced by at least two sources of Publicly Available Information, e.g. a news article on Reuters, the Wall Street Journal, the Financial Times or some other recognised publication." The delivery date is called the **Event Determination Date**.

**Step 2 — Notice of Physical Settlement:** "Within 30 calendar days of the Event Determination Date, the protection buyer must give Notice of Physical Settlement." This is called the **Conditions of Payment date**. The notice specifies what deliverable obligations will be delivered. Importantly, "this notice may be updated at any time between the Conditions of Payment date and the Physical Settlement date, with the last one being binding" (ISDA 2003).

**Step 3 — Physical Delivery:** "Within 30 business days of the Conditions of Payment date, the protection buyer must effect the physical delivery in return for par."

O'Kane notes that the entire process can extend significantly: "It is worth noting that there are up to 72 calendar days after initial notification until a payment on the protection leg must be made."

### 39.2.3 The Sourcing Problem and Short Squeezes

Physical settlement creates a practical problem when CDS notional exceeds deliverable bond supply. O'Kane describes the risk: "In some circumstances, the amount of outstanding protection that has been written which specifies physical settlement can be as large or even larger than the face value of deliverable obligations. This can also lead to a 'short squeeze' as protection buyers who do not hold a deliverable cash bond attempt to buy deliverable obligations to deliver into the contract."

The consequence is economically significant: "this demand can increase the price of the defaulted assets, thereby reducing the value of the loss payment." Protection buyers who need to source bonds in a squeezed market may pay inflated prices, reducing their effective protection.

Hull provides a concrete example: "When Lehman defaulted in September 2008, there was about $400 billion of CDS contracts and $155 billion of Lehman debt outstanding." Physical settlement for all contracts was clearly impossible.

---

## 39.3 Cash Settlement

### 39.3.1 Mechanics

O'Kane describes cash settlement: "The protection seller pays the protection buyer the face value of the protection minus the recovery price of the reference obligation in cash."

The payoff formula is:

$$\boxed{\Pi_{\text{cash}} = N\left(1 - \frac{P_{\text{final}}}{100}\right) = N(1 - R_{\text{settle}})}$$

where $P_{\text{final}}$ is the "final price" or "recovery price" (per 100 face) and $R_{\text{settle}} = P_{\text{final}}/100$ is the implied recovery fraction.

**Sanity checks:**
- If $P_{\text{final}} = 100$ (no loss): $\Pi_{\text{cash}} = 0$ ✓
- If $P_{\text{final}} = 0$ (total loss): $\Pi_{\text{cash}} = N$ ✓

> **Analogy: The Mechanic's Choice (Physical vs Cash)**
>
> You crashed your car (Bond). You have insurance (CDS).
>
> *   **Physical Settlement ("Here's the keys")**: You hand the wrecked car to the insurer. The insurer hands you a check for the *original full value* ($100).
> *   **Cash Settlement ("Keep the wreck")**: You keep the wrecked car (worth $30). The insurer sends you a check for the difference ($70).
>
> *Result*: In both cases, you end up with $100 value ($100 cash OR $30 wreck + $70 cash). But Physical requires you to actually *own* the car to hand it over.

### 39.3.2 How the Final Price Is Determined

The critical question for cash settlement is: who decides $P_{\text{final}}$, and how?

O'Kane states: "The recovery price is determined by a dealer poll or auction process."

Hull elaborates: "If, as is now usual, there is cash settlement, an ISDA-organized auction process is used to determine the mid-market value of the cheapest deliverable bond several days after the credit event."

The auction mechanism is covered in detail in Chapter 40. For this chapter, the key point is that cash settlement requires an external price determination process, and that price is designed to reflect the market value of deliverable obligations.

### 39.3.3 When Physical and Cash Settlement Align

If the cash settlement final price equals the market value of the obligation that would optimally be delivered under physical settlement, then both methods produce the same payout:

$$P_{\text{final}} = P_{\text{CTD}} \implies \Pi_{\text{cash}} = \Pi_{\text{phys}}$$

This alignment is the economic rationale for using "the mid-market value of the cheapest deliverable bond" in auctions—it attempts to replicate what physical settlement would achieve without requiring actual delivery.

### 39.3.4 Why Cash Settlement Became Standard

O'Kane explains the market evolution: "The current market standard for default swaps is to prefer physical settlement over cash settlement. However... there is a trend towards cash settlement for CDS, especially in the CDS index and STCDO markets."

The reason is practical: "The allowance for cash settlement is intended to overcome the problem of sourcing physical assets to deliver, especially when the amount of credit default protection sold is similar to or greater than the outstanding notional of deliverable obligations."

Following the Lehman example, cash settlement via auction has become the dominant mechanism for most CDS contracts.

> **Deep Dive: The Auction Two-Step**
>
> How do we find the "Final Price" ($P$) for billions of dollars of CDS? We can't just call one dealer.
>
> 1.  **Stage 1 (Initial Market Midpoint)**: Dealers quote two-way prices to buy/sell the bonds. This sets the initial "Inside Market" price.
> 2.  **Stage 2 (Limit Orders)**: Banks and Investors submit "Limit Orders" to buy or sell the bonds at the auction clearing price.
> 3.  **Clearing**: The price ($P$) is set where the net buy/sell interest clears. If everyone wants to sell bonds, the price drops until buyers step in.

---

## 39.4 The Delivery Option and Cheapest-to-Deliver

### 39.4.1 The Basket of Deliverables

A single CDS contract does not reference a single bond. O'Kane explains: "Typically, there can be many bonds and loans which qualify as deliverable obligations and so can be delivered into the default swap. The advantage of defining a basket of deliverables is that we can use one standard contract to hedge any of the bonds or loans which satisfy the criteria for a deliverable obligation. This removes the need to have different contracts for individual securities and so enhances liquidity."

O'Kane specifies the key constraint: "All deliverable obligations must be pari passu or senior to the reference obligation."

### 39.4.2 The Delivery Option

The basket creates embedded optionality. O'Kane states: "A side-effect of this is the protection buyer's right to choose the deliverable asset which means that the protection buyer is effectively long a delivery option."

Because the buyer chooses what to deliver, they will rationally deliver the obligation with the lowest market price—the **cheapest-to-deliver** (CTD):

$$P_{\text{CTD}} = \min_{i \in \mathcal{D}} P_i$$

where $\mathcal{D}$ is the set of eligible deliverables and $P_i$ is the market price of deliverable $i$.

The physical settlement payoff with optimal delivery is:

$$\boxed{\Pi_{\text{phys}} = N\left(1 - \frac{P_{\text{CTD}}}{100}\right)}$$

### 39.4.3 Delivery Option Value: A Worked Example

O'Kane provides a concrete illustration:

> "Consider a protection buyer holding $100 face value of an asset which trades at a price of $43 and $100 of protection. Suppose that a restructuring credit event occurs and that another deliverable asset is trading at a price of $37. To maximise his profit, the protection buyer can sell the original asset for $43, buy the cheaper asset for $37, and deliver that for par, making an additional profit of **$6 from the delivery option**."

This $6 (per 100 face) is pure profit from the ability to switch deliverables. The delivery option value is exactly the difference between the hedged asset's price and the CTD price:

$$\text{Delivery Option Value} = P_{\text{hedged}} - P_{\text{CTD}} = 43 - 37 = 6$$

### 39.4.4 When the Delivery Option Has Value

O'Kane explains when the option matters: "This option only has value if different deliverables trade at different prices following a credit event. We would therefore expect price differences to occur only for a soft credit event like restructuring since after a soft credit event, the assets of the reference entity can continue to trade with a term structure."

After a **hard** credit event (bankruptcy), all obligations trade at approximately the same recovery price, so the delivery option has minimal value.

After a **soft** credit event (restructuring), short-dated bonds trade higher than long-dated bonds, and high-coupon bonds trade higher than low-coupon bonds. The delivery option can be substantial.

### 39.4.5 Why CTD Matters for Protection Sellers

The delivery option is a zero-sum transfer from seller to buyer. The seller receives par for whatever the buyer chooses to deliver—and the buyer will choose the cheapest. From the seller's perspective, they are **short the delivery option**.

This asymmetry is captured in the spread ordering across contracts with different deliverability rules (Section 39.5).

---

## 39.5 Restructuring Clauses and the Market's Response

### 39.5.1 The Conseco Case

O'Kane documents how the delivery option's value became apparent:

> "The concerns of protection sellers were shown to be real following the restructuring of loans of the US insurer Conseco, Inc. in September 2000. Holders of default swap protection on Conseco triggered the protection and using the ISDA 1999 definitions on deliverable obligations, they were allowed to deliver long maturity and hence deep discount bonds trading in the 65-80 range in return for par. At the same time, shorter dated loans which were held by banks were trading at higher prices. It was claimed that banks were able to exploit the delivery option by selling their short dated loan assets, and buying the longer dated assets to deliver into the protection. In doing so they were able to receive more than the loss on the loans they were using the CDS to hedge."

This case demonstrated that the delivery option had real economic value—protection buyers received more than their actual hedged loss by exploiting the deliverable basket.

### 39.5.2 The Four Restructuring Clauses

O'Kane describes how the market responded by developing different restructuring treatments to limit the delivery option. Table 5.2 outlines the four main choices:

| Clause | Short Name | Description |
|--------|------------|-------------|
| **Old-Restructuring** | Old-Re (OR) | The original standard; maximum maturity deliverable is 30 years. |
| **Modified Restructuring** | Mod-Re (MR) | US standard since May 2001. Limits deliverable maturity to the maximum of remaining CDS maturity and minimum of 30 months/longest restructured obligation maturity. Only applies when triggered by protection buyer. |
| **Modified-Modified Restructuring** | Mod-Mod-Re (MMR) | European standard since 2003. Limits maturity to maximum of remaining CDS maturity and 60 months for restructured obligations / 30 months for non-restructured. Allows conditionally transferable obligations. Only applies when triggered by buyer. |
| **No-Restructuring** | No-Re (NR) | Removes restructuring as a credit event entirely. |

### 39.5.3 The Spread Ordering

O'Kane derives the logical spread relationship across these clauses:

$$\boxed{S_{\text{Old-Re}} > S_{\text{Mod-Mod-Re}} > S_{\text{Mod-Re}} > S_{\text{No-Re}}}$$

The reasoning is elegant: "The protection buyer who is long the delivery option should expect to have to pay for this optionality by paying a slightly higher spread than if the contract had no delivery option. The value of the delivery option is linked to the breadth of the range of deliverables."

- **Old-Re** has the widest deliverable basket (30-year maturity limit) → highest spread
- **Mod-Mod-Re** is broader than Mod-Re (60 months vs 30 months for restructured obligations)
- **Mod-Re** restricts maturities significantly
- **No-Re** eliminates restructuring entirely → lowest spread (no restructuring protection, no delivery option on restructuring)

### 39.5.4 Regional Differences: CDX vs iTraxx

O'Kane highlights an important index-level distinction: "While the European iTraxx indices include a restructuring credit event, the North American CDX index protection leg is only triggered by a bankruptcy or failure to pay on a reference credit. Restructuring is not included as a credit event. The CDX is therefore consistent with the No-Re category of CDS."

This creates a systematic difference between US and European credit index contracts:
- **CDX (North America):** No-Re — bankruptcy and failure to pay only
- **iTraxx (Europe):** Includes restructuring

When comparing index spreads across regions, this documentation difference must be considered.

---

## 39.6 Accrued Premium at Default

When a credit event occurs between premium payment dates, the protection buyer owes premium for the portion of the current period during which they had protection.

O'Kane confirms: "In both cases there will also be a payment of the coupon accrued at default from the protection buyer to the protection seller. This is, however, handled separately as it is part of the premium leg."

Hull provides an example: "In our example, where there is a default on May 20, 2023, the buyer would be required to pay to the seller the amount of the annual payment accrued between March 20, 2023, and May 20, 2023 (approximately $150,000), but no further payments would be required."

For a contract with annual spread $s$, notional $N$, and accrual fraction $\alpha$ since the last payment date:

$$\boxed{\text{Accrued Premium} = N \cdot s \cdot \alpha}$$

This is separate from the protection payout and represents the buyer's obligation for protection coverage during the partial period.

> **I'm not sure** about the exact day-count convention for accrued premium at default without the specific contract documentation. The sources confirm the existence and direction of this payment but do not fully specify conventions in the excerpts retrieved.

---

## 39.7 Settlement Risks

### 39.7.1 Final Price Uncertainty (Cash Settlement)

Under cash settlement, the protection payout is uncertain until the auction or dealer poll determines $P_{\text{final}}$. The payout sensitivity to the final price is:

$$\frac{\partial \Pi_{\text{cash}}}{\partial P_{\text{final}}} = -\frac{N}{100}$$

Each point change in the final price changes the payout by 1% of notional. For a $10 million position, a 10-point uncertainty in final price represents $1 million of payout uncertainty.

### 39.7.2 Deliverable Scarcity (Physical Settlement)

As discussed in Section 39.2.3, when CDS notional exceeds deliverable supply, protection buyers face sourcing risk. The short squeeze dynamic can push up deliverable prices, reducing effective protection.

### 39.7.3 Technical Default and Basis Risk

O'Kane lists "technical default" as a driver of the CDS-cash basis: "The standard credit events may be viewed as being broader than those which constitute default on a bond. As a result, protection sellers in a CDS may demand a higher spread as compensation for the increased risk of the protection being triggered."

This means a CDS may trigger when a bond does not technically default, or vice versa—creating basis risk between CDS and bond positions even when hedging the same credit.

---

## 39.8 Worked Examples

### Example A: Cash Settlement Payout

**Given:** $N = \$10\text{ million}$, $P_{\text{final}} = 38$.

**Step 1:** Convert price to recovery fraction:
$$R_{\text{settle}} = \frac{38}{100} = 0.38$$

**Step 2:** Compute payout:
$$\Pi_{\text{cash}} = 10{,}000{,}000 \times (1 - 0.38) = 10{,}000{,}000 \times 0.62 = \$6{,}200{,}000$$

**Answer:** The protection buyer receives **$6.2 million**.

---

### Example B: Physical Settlement Economics

**Given:** Deliverable bond trades at 38 per 100, $N = \$10\text{ million}$.

**Step 1:** Cost to source deliverable:
$$\text{Cost} = 10{,}000{,}000 \times \frac{38}{100} = \$3{,}800{,}000$$

**Step 2:** Cash received from seller:
$$\text{Cash} = \$10{,}000{,}000 \text{ (par)}$$

**Step 3:** Economic gain:
$$\Pi_{\text{phys}} = 10{,}000{,}000 - 3{,}800{,}000 = \$6{,}200{,}000$$

**Answer:** Same as Example A when $P_{\text{final}} = P_{\text{deliverable}} = 38$. This illustrates the alignment between physical and cash settlement.

---

### Example C: CTD with Multiple Deliverables

**Given:** Two eligible deliverables with prices $P_1 = 35$, $P_2 = 45$. Notional $N = \$10\text{ million}$.

**Step 1:** Identify CTD:
$$P_{\text{CTD}} = \min(35, 45) = 35$$

**Step 2:** Physical settlement payoff (optimal delivery):
$$\Pi_{\text{phys}} = 10{,}000{,}000 \times \left(1 - \frac{35}{100}\right) = 10{,}000{,}000 \times 0.65 = \$6{,}500{,}000$$

**Comparison with cash settlement at different prices:**
- Cash at 35: $\Pi = \$6.5\text{ million}$ (matches physical CTD)
- Cash at 45: $\Pi = 10{,}000{,}000 \times 0.55 = \$5.5\text{ million}$
- Cash at 40 (average): $\Pi = 10{,}000{,}000 \times 0.60 = \$6.0\text{ million}$

**Takeaway:** The protection buyer's choice to deliver the CTD increases their payout by $1 million compared to the higher-priced deliverable.

---

### Example D: Delivery Option Value (O'Kane Example)

**Given:** Buyer holds asset trading at 43, alternative deliverable trades at 37. Face value $100.

**Strategy:**
1. Sell held asset: receive $43
2. Buy CTD: pay $37
3. Deliver CTD for par: receive $100

**P&L decomposition:**
- Protection payout: $100 - 37 = 63$ (par minus CTD value)
- Net position from switch: $43 - 37 = 6$ (sold high, bought low)
- Delivery option value: **$6**

If buyer had simply delivered their held asset, they would have received $100 - 43 = 57$. By switching, they capture an additional $6.

---

### Example E: Accrued Premium at Default

**Given:** Annual spread $s = 200$ bp $= 0.02$, quarterly payment frequency, default occurs 45 days into a 90-day period. Notional $N = \$10\text{ million}$.

**Step 1:** Accrual fraction:
$$\alpha = \frac{45}{360} = 0.125 \text{ (using ACT/360-like logic)}$$

**Step 2:** Accrued premium:
$$\text{Accrued} = 10{,}000{,}000 \times 0.02 \times 0.125 = \$25{,}000$$

**Answer:** The protection buyer owes **$25,000** in accrued premium to the seller.

---

### Example F: Deliverable Set Restriction Sensitivity

**Given:** Original deliverables at prices 35 and 45. After reviewing documentation, the 35-priced bond is ruled ineligible. $N = \$10\text{ million}$.

**Original CTD payoff:** $10{,}000{,}000 \times 0.65 = \$6.5\text{ million}$

**New CTD payoff (only 45 eligible):** $10{,}000{,}000 \times 0.55 = \$5.5\text{ million}$

**Impact:** Payout decreases by **$1.0 million**.

**Interpretation:** Deliverability rules have material P&L impact. Understanding exactly what qualifies as a deliverable obligation is essential.

---

### Example G: Lehman Default (Hull)

**Facts:** Lehman bankruptcy September 2008. $400 billion CDS outstanding, $155 billion debt. Auction final price: 8.625%.

**Implied payout per $100 notional:**
$$\Pi = 100 \times (1 - 0.08625) = \$91.375$$

**For a $10 million position:**
$$\Pi = 10{,}000{,}000 \times 0.91375 = \$9{,}137{,}500$$

**Observation:** The extremely low recovery (8.625 cents on the dollar) produced near-maximal protection payouts.

---

## Practical Notes

### Mechanics Checklist for a Real Trade

Before modeling settlement outcomes, verify:

1. **Reference entity** and reference obligation(s)
2. **Documentation set** — ISDA Definitions vintage (2003? 2014?)
3. **Credit events included** — which events trigger; is restructuring included?
4. **Restructuring clause** — Old-Re / Mod-Re / Mod-Mod-Re / No-Re
5. **Settlement method** — physical, cash, or fallback language
6. **If physical:** deliverable obligation criteria and any maturity restrictions
7. **If cash:** how $P_{\text{final}}$ is determined (auction protocol)
8. **Notional, currency, settlement conventions**
9. **Accrued premium handling**

### Common Pitfalls

1. **Confusing "recovery" concepts:** The settlement final price is a market-determined number around the event, not a long-run modeling parameter.

2. **Assuming CTD is irrelevant under cash settlement:** Hull explicitly states auctions target "the mid-market value of the cheapest deliverable bond"—CTD economics are embedded in cash settlement.

3. **Forgetting accrued premium at default:** Premium is paid in arrears; a partial period accrual is owed even if the buyer never receives a full premium payment in that period.

4. **Mixing price conventions:** 38 (per 100) vs 0.38 (fraction) → factor of 100 payout error.

5. **Ignoring regional documentation differences:** CDX (No-Re) vs iTraxx (includes restructuring) is a meaningful distinction.

### Verification Tests for Risk Systems

1. **Payout bounds:** $0 \leq \Pi_{\text{cash}} \leq N$ when $0 \leq P_{\text{final}} \leq 100$
2. **Physical/cash alignment:** If $P_{\text{final}} = P_{\text{CTD}}$, then $\Pi_{\text{cash}} = \Pi_{\text{phys}}$
3. **CTD logic:** Under physical, buyer delivers lowest-priced eligible deliverable

---

## Summary

1. A CDS protection payment is triggered by a **credit event**—a contract-defined legal trigger that may differ from bond default definitions.

2. Credit events are classified as **hard** (bankruptcy, failure to pay) or **soft** (restructuring). Hard events cause all debt to trade at one price; soft events preserve a term structure.

3. **Physical settlement** involves delivering bonds for par; **cash settlement** pays $N(1 - P_{\text{final}}/100)$ using a dealer-determined price.

4. The **delivery option** arises because the buyer can choose which eligible obligation to deliver—they will choose the **cheapest-to-deliver** (lowest price).

5. The delivery option is most valuable after **restructuring** when different bonds trade at different prices.

6. **Restructuring clauses** (Old-Re, Mod-Re, Mod-Mod-Re, No-Re) evolved to limit delivery option value, with spread ordering reflecting optionality value.

7. **CDX excludes restructuring** (No-Re), while **iTraxx includes it**—a key regional difference.

8. **Deliverable scarcity** can cause short squeezes under physical settlement; auctions address this by providing cash settlement at a market-determined price.

9. **Accrued premium at default** is owed by the buyer for the partial period—handled as part of the premium leg.

10. Always verify the specific contract documentation; conventions evolve and "I'm not sure" is appropriate when sources don't specify.

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| Credit event | Contractual trigger for protection leg (may differ from bond default) | Determines whether you receive protection |
| Hard vs soft event | Hard: all debt trades at one price; Soft: term structure persists | Affects delivery option value |
| Physical settlement | Deliver bonds, receive par | Requires sourcing deliverables |
| Cash settlement | Receive $N(1 - P_{\text{final}}/100)$ | Requires price determination mechanism |
| Delivery option | Buyer's right to choose deliverable | Creates embedded optionality for buyer |
| CTD | Lowest-priced eligible deliverable | Maximizes buyer's payout |
| Restructuring clause | Limits on deliverables after restructuring | Controls delivery option value |
| Accrued premium | Partial premium for coverage until credit event | Owed by buyer at default |

---

## Notation for This Chapter

| Symbol | Definition |
|--------|------------|
| $N$ | CDS notional (USD) |
| $P_{\text{final}}$ | Final price (per 100) for cash settlement |
| $R_{\text{settle}} = P_{\text{final}}/100$ | Settlement-implied recovery fraction |
| $\mathcal{D}$ | Set of eligible deliverable obligations |
| $P_i$ | Market price (per 100) of deliverable $i$ |
| $P_{\text{CTD}} = \min_{i \in \mathcal{D}} P_i$ | Cheapest-to-deliver price |
| $\Pi_{\text{cash}}, \Pi_{\text{phys}}$ | Protection payout under cash/physical settlement |
| $\alpha$ | Accrual fraction (year fraction) |
| $s$ | CDS contractual spread (annualized decimal) |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is a "credit event" in CDS terms? | A contract-defined legal trigger for protection leg payment; similar to default but not identical to rating-agency definitions. |
| 2 | What are the two settlement methods for CDS? | Physical settlement (deliver bonds, receive par) and cash settlement (receive $N(1-P_{\text{final}}/100)$). |
| 3 | What distinguishes hard from soft credit events? | Hard events (bankruptcy) cause all debt to trade at one price; soft events (restructuring) preserve term structure. |
| 4 | Which is the only soft credit event? | Restructuring. |
| 5 | What is the delivery option? | The buyer's right to choose which deliverable to deliver—creates embedded optionality. |
| 6 | What is CTD in CDS settlement? | Cheapest-to-deliver: the lowest-priced eligible deliverable obligation. |
| 7 | Why does the delivery option favor the buyer? | Buyer delivers the cheapest obligation and receives par, maximizing payout. |
| 8 | Cash settlement payoff formula? | $\Pi_{\text{cash}} = N(1 - P_{\text{final}}/100)$ |
| 9 | Physical settlement payoff with CTD? | $\Pi_{\text{phys}} = N(1 - P_{\text{CTD}}/100)$ |
| 10 | When is the delivery option most valuable? | After restructuring (soft event) when deliverables trade at different prices. |
| 11 | What is a short squeeze in this context? | When CDS notional exceeds deliverable supply, buying pressure raises deliverable prices, reducing protection value. |
| 12 | What was the Lehman auction recovery price? | 8.625% of face value (91.375% payout). |
| 13 | Name the four restructuring clauses. | Old-Re, Mod-Re, Mod-Mod-Re, No-Re. |
| 14 | What is the spread ordering across restructuring clauses? | $S_{\text{Old-Re}} > S_{\text{Mod-Mod-Re}} > S_{\text{Mod-Re}} > S_{\text{No-Re}}$ |
| 15 | What clause does CDX use? | No-Re (excludes restructuring as credit event). |
| 16 | What clause does iTraxx use? | Includes restructuring (Mod-Mod-Re style). |
| 17 | What triggered the Mod-Re introduction? | The Conseco restructuring (September 2000) where delivery option was exploited. |
| 18 | What is accrued premium at default? | Partial premium owed for coverage from last payment date to credit event. |
| 19 | Who owes accrued premium at default? | Protection buyer owes it to seller. |
| 20 | If $P_{\text{final}} = 100$, what is the payout? | Zero (no loss). |
| 21 | If $P_{\text{final}} = 0$, what is the payout? | $N$ (total loss). |
| 22 | How much profit did O'Kane's delivery option example show? | $6 per $100 face (sell $43 asset, buy $37 CTD, deliver for par). |
| 23 | What determines $P_{\text{final}}$ in cash settlement? | Dealer poll or ISDA auction process. |
| 24 | What does the auction target according to Hull? | "The mid-market value of the cheapest deliverable bond." |
| 25 | Key criterion for deliverable obligations? | Must be pari passu or senior to the reference obligation. |
| 26 | Maximum physical settlement timeline? | Up to 72 calendar days after initial notification. |
| 27 | What is "technical default" in CDS-bond basis? | CDS credit events may be broader than bond default, creating basis risk. |
| 28 | When do physical and cash settlement give the same payout? | When $P_{\text{final}} = P_{\text{CTD}}$. |
| 29 | How much CDS was outstanding on Lehman vs actual debt? | $400B CDS vs $155B debt. |
| 30 | Why did cash settlement become standard? | To avoid sourcing problems when CDS notional exceeds deliverable supply. |

---

## Mini Problem Set

1. A CDS has $N = \$25\text{mm}$ and auction final price 12. Compute the cash settlement payout.

2. For $N = \$10\text{mm}$, compare payouts if final price is 38 vs 42. What is the difference?

3. Two deliverables are priced at 55 and 60. Under physical settlement with $N = \$10\text{mm}$, what is the CTD and payout?

4. Three deliverables priced 20, 35, 50 with $N = \$10\text{mm}$. Compute physical settlement payout. Compare to cash at $P_{\text{final}} = 35$.

5. In Q4, the 20-priced bond is ruled ineligible. Recompute the payout and the change.

6. Explain why a soft credit event creates larger delivery option value than a hard event.

7. A buyer holds a bond at 48 with CDS, and CTD after restructuring is 40. Per $100 face, what is the delivery option value? For $N = \$10\text{mm}$?

8. Spread $s = 400$ bp, default 60 days into a 90-day period, $N = \$20\text{mm}$. Estimate accrued premium (state assumptions).

9. Derive: if $P_{\text{final}} = P_{\text{CTD}}$, then $\Pi_{\text{cash}} = \Pi_{\text{phys}}$.

10. CTD bond has bid/ask of 25/35 in distressed market. For $N = \$50\text{mm}$, compute the payout range.

---

### Solution Sketches (Selected)

**Q1:** $\Pi = 25\text{mm} \times (1 - 0.12) = 25\text{mm} \times 0.88 = \$22\text{mm}$

**Q2:** At 38: $\Pi = 10\text{mm} \times 0.62 = \$6.2\text{mm}$. At 42: $\Pi = 10\text{mm} \times 0.58 = \$5.8\text{mm}$. Difference: $\$0.4\text{mm}$.

**Q3:** CTD = 55. $\Pi = 10\text{mm} \times (1 - 0.55) = 10\text{mm} \times 0.45 = \$4.5\text{mm}$

**Q4:** Physical uses CTD = 20: $\Pi = 10\text{mm} \times 0.80 = \$8.0\text{mm}$. Cash at 35: $\Pi = 10\text{mm} \times 0.65 = \$6.5\text{mm}$. Physical payout is $\$1.5\text{mm}$ higher due to CTD.

**Q5:** New CTD = 35: $\Pi = 10\text{mm} \times 0.65 = \$6.5\text{mm}$. Change: $-\$1.5\text{mm}$ from Q4.

**Q6:** Soft events preserve term structure → different obligations trade at different prices → buyer can choose cheapest. Hard events collapse all debt to one price → no choice benefit.

**Q7:** Delivery option value = $48 - 40 = \$8$ per $100. For $\$10\text{mm}$: $10{,}000{,}000 \times 0.08 = \$800{,}000$.

**Q9:** $\Pi_{\text{cash}} = N(1 - P_{\text{final}}/100)$. $\Pi_{\text{phys}} = N(1 - P_{\text{CTD}}/100)$. If $P_{\text{final}} = P_{\text{CTD}}$, both equal $N(1 - P_{\text{CTD}}/100)$. QED.

---

## Source Map

### (A) Verified Facts (Source-Backed)

| Fact | Source |
|------|--------|
| Credit event is the legal trigger for protection leg; differs from rating-agency default | O'Kane 5.4.1 |
| Credit event types table (six events with hard/soft classification) | O'Kane Table 5.1 |
| Restructuring is the only soft credit event | O'Kane 5.4.1 |
| After restructuring, debt trades with term structure of prices | O'Kane 5.4.1 |
| Physical settlement: deliver face value, receive par in cash | O'Kane 5.4 |
| Cash settlement: seller pays $(\text{face} - \text{recovery price})$; recovery determined by poll/auction | O'Kane 5.4 |
| Deliverable obligations must be pari passu or senior to reference obligation | O'Kane 5.4 |
| Delivery option: buyer can choose which deliverable to deliver | O'Kane 5.4.3 |
| O'Kane delivery option example: $43 asset, $37 CTD → $6 profit | O'Kane 5.4.3 |
| Conseco restructuring (Sept 2000): banks delivered deep-discount bonds (65-80) for par | O'Kane 5.4.3 |
| Restructuring clauses: Old-Re, Mod-Re, Mod-Mod-Re, No-Re with descriptions | O'Kane Table 5.2 |
| Spread ordering: $S_{\text{Old-Re}} > S_{\text{Mod-Mod-Re}} > S_{\text{Mod-Re}} > S_{\text{No-Re}}$ | O'Kane 5.4.4 |
| CDX uses No-Re; iTraxx includes restructuring | O'Kane Ch 10 |
| Physical settlement timeline: up to 72 calendar days | O'Kane 5.4.2 |
| Credit Event Notice requires two sources of Publicly Available Information | O'Kane 5.4.2 |
| Notice of Physical Settlement: within 30 calendar days, can be updated until delivery | O'Kane 5.4.2 (ISDA 2003) |
| Short squeeze when CDS notional exceeds deliverable supply | O'Kane 5.4.2 |
| Cash settlement auction targets "mid-market value of cheapest deliverable bond" | Hull Ch 25 |
| Lehman: $400B CDS, $155B debt; auction payout 91.375% (8.625% recovery) | Hull Ch 25 |
| Accrued premium at default paid by buyer to seller | O'Kane 5.4, Hull Ch 25 |
| Technical default as CDS-bond basis driver | O'Kane 5.6 |

### (B) Reasoned Inference (Derived from A)

- **CTD payoff formula:** $\Pi_{\text{phys}} = N(1 - P_{\text{CTD}}/100)$ — derived from physical settlement mechanics (buyer delivers cheapest, receives par)
- **Cash/physical alignment condition:** When $P_{\text{final}} = P_{\text{CTD}}$, both methods yield same payout — follows from payoff formulas
- **Delivery option value:** $P_{\text{hedged}} - P_{\text{CTD}}$ — explicitly stated by O'Kane
- **Payout bounds:** $0 \leq \Pi \leq N$ for $0 \leq P \leq 100$ — arithmetic from payoff formula

### (C) Flagged Uncertainties

- **Auction operational details:** I'm not sure about the precise two-stage auction mechanics, bidding rules, timing, and Determinations Committee procedures from the excerpts retrieved. Chapter 40 covers auctions in more detail; for complete protocol, the specific ISDA documentation is required.
- **Complete deliverability criteria:** I'm not sure of all eligibility criteria beyond "pari passu or senior" without the precise ISDA Definitions and confirmation terms.
- **Accrual day-count conventions:** I'm not sure of the exact day-count convention for accrued premium at default without specific contract documentation.
- **Current Determinations Committee rules:** The sources reference ISDA 2003 definitions; current procedures may have evolved.
