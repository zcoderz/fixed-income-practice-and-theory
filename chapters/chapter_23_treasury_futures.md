# Chapter 23: Treasury Futures — CTD, Conversion Factors, Delivery Options, and the Link to Repo

---

## Introduction

The Treasury bond futures contract doesn't specify which bond you deliver—so which one do you choose?

This seemingly simple question reveals the distinctive feature that makes bond futures pricing both fascinating and challenging. Unlike most futures contracts where the underlying asset is precisely defined, Treasury futures allow the short position to deliver any bond from a specified basket. The short can also choose *when* to deliver within an extended window. These embedded options—the **quality option** to choose the bond and the **timing option** to choose the date—transfer value from the long to the short and depress the futures price below what a simple cost-of-carry calculation would suggest.

Understanding this mechanism is essential for anyone trading, hedging, or pricing with Treasury futures. A hedger who ignores the delivery option dynamics may find their carefully constructed DV01 hedge suddenly misaligned when the cheapest-to-deliver bond switches. A basis trader who focuses only on gross basis without accounting for carry and option value will systematically misprice the trade's expected return. As Hull notes explicitly: "An exact theoretical futures price for the Treasury bond contract is difficult to determine because the short party's options concerned with the timing of delivery and choice of the bond that is delivered cannot easily be valued."

For middle-office professionals transitioning to front office roles, Treasury futures present both opportunity and risk. The P&L reports you already see contain basis P&L, roll P&L, and CTD switch impacts—but understanding what drives those numbers requires mastering the concepts in this chapter. When a hedge ratio jumps by 15% overnight and you need to explain it to the desk head, the answer lies in CTD switching. When the roll spread blows out during quarter-end, the explanation involves squeeze risk and repo specialness.

This chapter covers:

1. **The deliverable basket and conversion factors** — how the contract standardizes delivery across bonds of different coupons and maturities
2. **Cost of delivery and the CTD** — how the short identifies the optimal bond to deliver
3. **When CTD switches** — how yield levels, curve shape, and financing conditions shift CTD
4. **Gross basis, net basis, and implied repo** — the metrics practitioners use for cash-futures relative value
5. **Delivery options and their value** — quality, timing, end-of-month, and wild card features
6. **The repo-futures link** — how financing costs enter through carry and specialness
7. **Hedging implications** — why futures DV01 depends on CTD and can jump
8. **Futures rolls and calendar spreads** — how practitioners manage rolling positions
9. **Squeeze risk** — what happens when the CTD becomes scarce
10. **Model selection** — when to use simple vs. multi-factor approaches

The mechanics developed here connect directly to Part II's bond pricing (Chapter 5), Part III's risk measures (Chapters 11–16), and Chapter 9's treatment of repo. The carry and forward-pricing relationships from Chapter 9 are essential background; this chapter focuses on what makes futures *different* from forwards.

---

## 23.1 Treasury Futures as a Deliverable-Basket Contract

### 23.1.1 The Contract Structure

Treasury futures obligate the short to deliver Treasury securities of a specified maturity range during the delivery month. The contract specifies a **delivery basket**: a set of eligible bonds that the short may choose from. Tuckman describes these as "a basket of eligible Treasury securities" and notes that the short satisfies the contract by delivering one eligible bond of the face amount specified by the contract.

The design of bond futures contracts purposely avoids a single underlying security. Tuckman explains two key reasons for this choice: First, "if the single underlying bond should lose liquidity, perhaps because it has been accumulated over time by buy-and-hold investors and institutions, then the futures contract would lose its liquidity as well." Second, a single deliverable creates the possibility of a **squeeze**, where a trader might "profit by simultaneously purchasing a large fraction of that bond issue and a large number of contracts," forcing shorts to buy that bond at distorted prices.

The contract face amount is typically $100,000. The short receives an **invoice price** based on the futures settlement price and a bond-specific adjustment called the **conversion factor**.

> **Key insight:** The futures is not a forward on a single bond. It is a forward-like commitment to exchange *some* Treasury security, with embedded options held by the short.

### 23.1.2 The Short's Embedded Options

Hull summarizes the delivery options available to the short: "In the Treasury bond futures contracts, the party with the short position has a number of interesting delivery options: (1) Delivery can be made on any day during the delivery month. (2) There are a number of alternative bonds that can be delivered. (3) On any day during the delivery month, the notice of intention to deliver at the 2:00 p.m. settlement price can be made later in the day." Hull then states explicitly: "These options all tend to reduce the futures price."

The short position holder has several embedded options:

**Quality option:** The short chooses which eligible bond to deliver. Different bonds in the basket have different prices, coupons, and risk characteristics. The short will naturally choose the bond that minimizes the cost of delivery. Tuckman notes this is "the most significant embedded option in futures contracts."

**Timing option:** The short can deliver on any day during the delivery month (or delivery window). Tuckman explicitly describes this: "The party short the futures contract may deliver at any time during the delivery month."

**End-of-month option:** In U.S. contracts, the last trade date typically precedes the last delivery date. After the last trade, the settlement price is fixed, but the short can still choose which day (and which bond) to deliver. If prices move, the short can switch to a more favorable bond. Tuckman notes this feature "gives rise to the end-of-month option."

**Wild card play:** Hull describes this timing feature in Business Snapshot 6.2: "The settlement price in the CME Group's Treasury bond futures contract is the price at 2:00 p.m. Chicago time. However, Treasury bonds continue trading in the spot market beyond this time and a trader with a short position can issue to the clearing house a notice of intention to deliver later in the day." If bond prices fall between 2:00 p.m. and the notice deadline, the short benefits by delivering at the higher (earlier) settlement price while buying bonds cheaper. Hull notes: "As with the other options open to the party with the short position, the wild card play is not free. Its value is reflected in the futures price, which is lower than it would be without the option."

> **Desk Reality: Why These Options Matter for P&L**
>
> If you've ever seen a futures position show unexplained P&L when rates barely moved, the delivery options may be the culprit. The embedded optionality means futures don't move 1:1 with their CTD—especially near delivery or when CTD is close to switching. Your risk system may show a clean DV01-neutral hedge, but the P&L says otherwise. The explanation often lies in the convexity effects from these options.

---

## 23.2 Conversion Factors: Standardizing the Delivery Basket

### 23.2.1 Why Conversion Factors Exist

Bonds in the delivery basket differ in coupon and maturity. Without any adjustment, a high-coupon bond would be much more valuable than a low-coupon bond of the same maturity. The futures contract would then essentially become a contract on the highest-coupon bond, since the short would never deliver anything else.

**Conversion factors** solve this problem by adjusting the delivery price for each bond's characteristics. Hull explains: "When a particular bond is delivered in a CME group bond futures contract, a parameter known as its conversion factor defines the price received for the bond by the party with the short position."

Tuckman provides the economic rationale for using a basket with conversion factors: "The large difference between delivering the CTD versus the next to CTD when all conversion factors are one arises because the same credit is given for delivering the low-coupon [bond] as the [high-coupon bond]. Actual conversion factors reduce the differences in delivery costs across bonds by adjusting delivery prices for coupon rates."

### 23.2.2 The 6% Rule: How Conversion Factors Are Calculated

The conversion factor is computed as the bond's clean price per $1 face value when discounted at a **notional yield of 6%** (with semiannual compounding), using contract-specific rounding rules.

Hull states: "The conversion factor for a bond is set equal to the quoted price the bond would have per dollar of principal on the first day of the delivery month on the assumption that the interest rate for all maturities equals 6% per annum (with semiannual compounding)."

**Hull's first worked example:** Consider a 10% coupon bond with 20 years and 2 months to maturity. For the purposes of calculating the conversion factor, the bond is assumed to have exactly 20 years to maturity (rounding down to the nearest 3 months for bond futures). The first coupon is assumed to be made after 6 months. Hull calculates:

$$\text{Value at 6\%} = \sum_{i=1}^{40} \frac{5}{1.03^{i}} + \frac{100}{1.03^{40}} = \$146.23$$

Dividing by the face value gives a conversion factor of **1.4623**.

**Hull's second worked example:** Consider an 8% coupon bond with 18 years and 4 months to maturity. For conversion factor purposes, the bond is assumed to have exactly 18 years and 3 months to maturity. Discounting all payments back to a point 3 months from today at 6% per annum (compounded semiannually) gives:

$$4 + \sum_{i=1}^{36} \frac{4}{1.03^{i}} + \frac{100}{1.03^{36}} = \$125.8323$$

The interest rate for a 3-month period is $\sqrt{1.03} - 1 = 1.4889\%$. Hence, discounting back to the present gives the bond's value as $125.8323 / 1.014889 = \$123.99$. Subtracting the accrued interest of 2.0 gives $\$121.99$. The conversion factor is therefore **1.2199**.

Tuckman frames the intuition: "The conversion factor of a bond is approximately equal to its price per dollar face amount as of the last delivery date with a yield equal to the notional coupon rate." He adds: "Conversion factors approximately equal the bond's price per $1 face value if yields were at the notional coupon rate; they would adjust prices perfectly if the term structure were flat at that notional rate."

> **Practitioner Note: Historical Context**
>
> The notional coupon for U.S. Treasury futures was historically 8% when the contracts were first introduced. In February 1999, the CBOT approved changing the notional coupon to 6%, effective with the March 2000 Treasury bond and Treasury note futures contracts. This historical context helps explain why some older references use 8% in their examples.

### 23.2.3 What Conversion Factors Are NOT

It's important to understand what conversion factors don't do:

- **Not a duration or DV01:** The conversion factor is a price-scaling parameter, not a sensitivity measure
- **Not a market price:** Conversion factors are fixed for each bond at contract inception and do not vary with market conditions
- **Not perfect:** When yields differ from 6%, conversion factors do not perfectly equalize delivery attractiveness across bonds

This imperfection—the fact that conversion factors only work perfectly at a flat 6% yield—creates the quality option's value.

---

## 23.3 Invoice Price and Cost of Delivery

### 23.3.1 The Invoice Price Formula

When bond $i$ is delivered into the futures contract, the short receives:

$$\boxed{\text{Invoice}_i(t) = cf_i \times F(t) + AI_i(t)}$$

where:
- $cf_i$ is the conversion factor for bond $i$
- $F(t)$ is the futures settlement price at time $t$
- $AI_i(t)$ is the accrued interest on bond $i$ at time $t$

Hull states explicitly: "The applicable quoted price for the bond delivered is the product of the conversion factor and the most recent settlement price for the futures contract. Taking accrued interest into account... the cash received for each $100 face value of the bond delivered is (Most recent settlement price × Conversion factor) + Accrued interest."

**Hull's numerical example:** Suppose the most recent settlement price is 120-00 (i.e., 120.00), the conversion factor for the bond delivered is 1.3800, and the accrued interest is $3 per $100 face value. The cash received is:

$$(1.3800 \times 120.00) + 3.00 = \$168.60$$

per $100 face value. A party with the short position in one contract would deliver bonds with face value of $100,000 and receive $168,600.

**Intuition:** The futures price is quoted like a clean price. The invoice adds accrued interest because the delivered bond exchanges at its dirty (cash) price.

### 23.3.2 Cost of Delivery

The **cost of delivery** measures what the short pays to buy the bond minus what they receive from delivering it. Tuckman gives this as equation (20.1):

$$\text{Cost of buying bond } i = P_i(t) + AI_i(t)$$

$$\text{Invoice received} = cf_i \times F(t) + AI_i(t)$$

$$\boxed{\text{CostDel}_i(t) = P_i(t) - cf_i \times F(t)}$$

Tuckman notes that accrued interest cancels because it appears in both the purchase cost and the invoice received.

**Unit check:** All terms are in dollars per $100 face. The cost of delivery is positive if the bond costs more to buy than the invoice received (i.e., it "costs" the short to deliver).

---

## 23.4 The Cheapest-to-Deliver (CTD) Bond

### 23.4.1 The CTD Criterion

The **cheapest-to-deliver** bond is the one that minimizes the cost of delivery—or equivalently, maximizes the short's profit from delivering.

$$\boxed{\text{CTD} = \arg\min_i \{P_i(t) - cf_i \times F(t)\}}$$

> **Analogy: The Fixed Price Menu**
>
> Imagine you are a short seller committed to buying dinner for the long holder.
> *   **The Contract**: You sold a "Dinner Futures" contract for $20. You must deliver *one* item from the menu.
> *   **The Menu**:
>     *   Burger: Cost $10. Adjusted Price = $10 / 1.0 = $10.
>     *   Steak: Cost $25. Adjusted Price = $25 / 2.0 = $12.50.
>     *   Salad: Cost $8. Adjusted Price = $8 / 0.9 = $8.88.
>
> *   **The Choice**: You will obviously buy the salad ($8) because it's the cheapest way to fulfill the contract, even though the contract price is fixed.
> *   **The Switch**: If salad prices skyrocket to $15, you will switch to buying the burger. This ability to *switch* is the "Quality Option." It's valuable to you (Short), and risky for the eater (Long).

**Example from Hull (Example 6.1):** A short considers three bonds with the most recent settlement price at 93-08 (93.25):

| Bond | Quoted Price | Conversion Factor | Cost of Delivery |
|------|--------------|-------------------|------------------|
| 1 | $99.50 | 1.0382 | $99.50 - 93.25 × 1.0382 = $2.69 |
| 2 | $143.50 | 1.5188 | $143.50 - 93.25 × 1.5188 = $1.87 |
| 3 | $119.75 | 1.2615 | $119.75 - 93.25 × 1.2615 = $2.12 |

Bond 2 has the lowest cost of delivery and is therefore CTD.

### 23.4.2 Final Settlement Price at Delivery

Tuckman shows (equation 20.3) that at the final delivery date, the settlement price must satisfy:

$$\boxed{F(T) = \min_i \frac{P_i(T)}{cf_i} = \frac{P^{\text{CTD}}(T)}{cf^{\text{CTD}}}}$$

This is proved by arbitrage. Tuckman shows: "First assume that $F(T) > P^{CTD}(T)/cf^{CTD}$. In this case a trader could buy the CTD, sell the contract, and deliver the CTD. The profit from this trade is $cf^{CTD} \times F(T) - P^{CTD}(T)$. But, by assumption, this is positive and, therefore, the trade constitutes an arbitrage opportunity."

Similarly, if $F(T) < P^{CTD}(T)/cf^{CTD}$, a trader could sell the CTD, buy futures, and take delivery—also creating arbitrage.

A key consequence: **the cost of delivering the CTD at final settlement is zero** (Tuckman equation 20.7):

$$P^{\text{CTD}}(T) - cf^{\text{CTD}} \times F(T) = 0$$

For any non-CTD bond $j$:

$$P^j(T) - cf^j \times F(T) > 0$$

The cost of delivering a non-CTD bond is positive.

---

## 23.5 Why CTD Can Switch: Yield Level and Curve Shape

### 23.5.1 The Imperfection of Conversion Factors

Conversion factors would perfectly equalize delivery attractiveness only if the term structure were flat at exactly 6%. In reality, yields differ from 6%, and the curve has shape. This creates systematic biases.

Tuckman explains the mechanism: "As yield moves away from the notional coupon it is no longer true that conversion factors perfectly adjust delivery prices." The key insight involves duration.

At a yield of 6%, the conversion factor approximately equals the bond's price per dollar face. Tuckman derives that the slope of the price/CF ratio with respect to yield is approximately proportional to the bond's duration:

$$\frac{d(P_i/cf_i)}{dy} \approx -\frac{P_i}{cf_i} \times D_i$$

Since at 6% the conversion factor equals price per dollar face, this slope is approximately proportional to duration.

### 23.5.2 Yield Level Effects

**When yields rise above 6%:** Higher-duration bonds fall more in price (since duration measures price sensitivity). But their conversion factors don't change. So higher-duration bonds become relatively cheaper to deliver.

Tuckman: "As yield increases above the notional coupon rate the prices of all bonds fall, but the price of the bond with the highest duration... falls the most."

**When yields fall below 6%:** Lower-duration bonds rise less in price, making them relatively cheaper to deliver.

Tuckman: "As yield falls below the notional coupon rate, the prices of all bonds increase but the price of the bond with the lowest duration... increases the least."

Hull summarizes the pattern: "When bond yields are in excess of 6%, the conversion factor system tends to favor the delivery of low-coupon long-maturity bonds. When yields are less than 6%, the system tends to favor the delivery of high-coupon short-maturity bonds."

### 23.5.3 CTD Tends Toward Extremes

This creates a systematic pattern:

| Yield Level | CTD Tendency |
|-------------|--------------|
| Yields well above 6% | Longest duration (low coupon, long maturity) |
| Yields near 6% | All bonds similar; intermediate duration |
| Yields well below 6% | Shortest duration (high coupon, short maturity) |

> **Visual: The CTD Switch Graph**
>
> Imagine plotting the Adjusted Price ($P/cf$) of three bonds against Yield.
>
> *   **Low Yield**: The Short Duration bond line is lowest. (CTD = Short).
> *   **High Yield**: The Long Duration bond line falls faster (steeper slope) and crosses below. (CTD = Long).
> *   **The Crossing Point**: This is where the lines intersect. At this exact yield, the short is indifferent.
> *   **The Futures Price**: It follows the *bottom envelope* of all these lines. This curved shape (concave) equals **Negative Convexity**.

### 23.5.4 Negative Convexity of Futures

A striking consequence of CTD switching is that Treasury futures exhibit **negative convexity**. Tuckman explains: "At any yield level the futures price at delivery is, according to equation (20.3), the ratio of the price of the CTD to its conversion factor. Graphically, the futures price is the lower envelope of the price ratio-yield curves."

As yields change and CTD switches, Tuckman notes: "As yield decreases the duration of the futures contract moves from resembling the relatively high duration of the [longest bond] to resembling the relatively low duration of the [shortest bond]. Hence, at delivery, the duration of the futures contract falls with yield; that is, the contract is negatively convex."

This negative convexity is a direct consequence of the quality option: the short always delivers the bond that minimizes the futures value, creating a "minimum envelope" effect.

### 23.5.5 Curve Shape Also Matters

With a non-flat curve, intermediate-maturity bonds can become CTD even when yields are not at 6%. Tuckman notes: "If general yield levels were to fall further, the CTD would shift to the front end of the basket. If the curve were to steepen more, the CTD would shift to the back end of the basket."

Hull also observes: "When the yield curve is upward-sloping, there is a tendency for bonds with a long time to maturity to be favored, whereas when it is downward-sloping, there is a tendency for bonds with a short time to maturity to be delivered."

The two-dimensional nature of CTD determination (yield level + curve shape) makes the futures contract's risk profile more complex than a simple bond position.

---

## 23.6 Gross Basis, Net Basis, and Carry

### 23.6.1 Gross Basis

The **gross basis** for bond $i$ at time $t$ is simply the cost of delivery (Tuckman equation 20.10):

$$\boxed{GB^i(t) = P^i(t) - cf^i \times F(t)}$$

A positive gross basis means the bond is "rich" to the futures after CF adjustment; negative means "cheap."

### 23.6.2 Net Basis and the Role of Carry

Gross basis ignores the cost of financing the bond until delivery. The **net basis** accounts for carry. Tuckman defines net basis (equation 20.11) as:

$$\boxed{NB^i(t) = P_{\text{fwd}}^i(t) - cf^i \times F(t)}$$

where $P_{\text{fwd}}^i(t)$ is the forward price of bond $i$ for delivery at $T$.

Using the carry relationship from Chapter 9 (and Tuckman equation 16.8), the forward price may be written in terms of spot price and carry:

$$P_{\text{fwd}} = P(0) - \text{Carry}$$

Substituting into the net basis definition:

$$\boxed{NB^i(t) = GB^i(t) - \text{Carry}^i(t \to T)}$$

Tuckman explicitly states (equation 20.12): "The right-hand side of equation (20.12) explains the term net basis: It is the gross basis net of carry."

**Why net basis matters:** At delivery, the forward price equals the spot price (carry = 0), so gross basis equals net basis, and both equal the cost of delivery. The CTD has zero cost of delivery at final settlement. Therefore, **net basis measures the value of the quality option** that can be locked in at time $t$.

Tuckman interprets: "If the net basis of any bond is near zero, then the quality option embedded in the contract is nearly worthless and selling that bond forward is equivalent to selling the futures contract."

### 23.6.3 Net Basis as Straddle-Like Exposure

Tuckman provides a key insight about net basis behavior for bonds close to CTD: "The net basis of a bond close to CTD behaves like a straddle on rates or prices." Rate moves in either direction push the bond away from CTD and increase its net basis. This straddle-like characteristic reflects the optionality embedded in the CTD mechanism.

> **Desk Reality: Reading Net Basis**
>
> When you see net basis near zero for a bond, it's telling you that bond is CTD and the quality option is nearly worthless—the futures is trading close to its theoretical floor. When net basis widens, it means the quality option has value: either CTD might switch, or the bond has moved away from CTD.
>
> For basis traders, net basis is the economic measure. Gross basis can be misleading because it ignores carry—a bond might look cheap on gross basis but expensive after accounting for financing costs.

### 23.6.4 Basis Trade P&L

Tuckman shows (equation 20.15) that a long basis trade (buy bond, finance in repo, short futures with proper tailing) has P&L proportional to the change in net basis:

$$\text{P\&L} = G^i \times [NB^i(t') - NB^i(t)]$$

where $G^i$ is the face amount of the bond position. In words, "the P&L from the long basis position equals the size of the bond position times the change in the net basis."

---

## 23.7 Implied Repo Rate

### 23.7.1 Definition

The **implied repo rate** is the financing rate at which a cash-and-carry trade (buy bond, finance in repo, deliver into futures) breaks even.

Setting the delivery profit to zero:

$$(P_i(0) + AI_i(0))(1 + r_{\text{imp}} \times d/360) = cf_i \times F(T) + AI_i(T)$$

Solving:

$$\boxed{r_{\text{imp},i} = \left(\frac{cf_i \times F(T) + AI_i(T)}{P_i(0) + AI_i(0)} - 1\right) \frac{360}{d}}$$

### 23.7.2 Interpretation

- **If $r_{\text{imp}} > r_{\text{repo}}$:** The trade earns more than the cost of financing. Cash-and-carry (long basis) is attractive.
- **If $r_{\text{imp}} < r_{\text{repo}}$:** The trade costs more to finance than it earns. Cash-and-carry is unattractive.

The CTD typically has an implied repo rate close to—or below—the actual repo rate, reflecting the quality option's value. For non-CTD bonds, implied repo is usually below market repo rates (the trade loses money unless that bond becomes CTD).

---

## 23.8 The Link to Repo and Specialness

### 23.8.1 Financing Cost Enters Through Carry

The connection between repo and futures pricing runs through carry. From Chapter 9, the forward price under repo financing is:

$$\boxed{P_{\text{fwd}} + AI(T) = (P(0) + AI(0))(1 + rd/360)}$$

Lower repo rates mean lower financing costs, which increases carry and reduces the forward price. This affects net basis and therefore CTD economics.

### 23.8.2 Specialness Can Shift CTD

Tuckman discusses how some bonds finance "special" (at repo rates below general collateral) due to high demand to borrow those specific securities.

**Special spread** = GC rate - Special rate

If a bond in the delivery basket can be financed special, its carry improves. This can make it CTD even if its cash price is slightly higher than alternatives.

**Example logic:** Bond A costs $101.20 to buy, but finances at 3.5% (special). Bond B costs $100.90 but finances at 5% (GC). The lower financing cost for A may more than offset its higher price, making A the CTD.

> **Cross-reference:** Chapter 9 develops the mechanics of special repo. Here we focus on its implications for futures pricing.

---

## 23.9 Delivery Options and Their Value

### 23.9.1 Why Delivery Options Depress the Futures Price

Hull states explicitly: "An exact theoretical futures price for the Treasury bond contract is difficult to determine because the short party's options concerned with the timing of delivery and choice of the bond that is delivered cannot easily be valued."

The delivery options all benefit the short. In a competitive market, the short "pays" for these options by accepting a lower futures price. The futures trades **below** what a simple cost-of-carry model would predict.

### 23.9.2 Quality Option Value

Tuckman interprets net basis as the quality option value: "The cost you can lock in is $P_{\text{fwd}}^i(t) - cf^i \times F(t)$, which is net basis... net basis is the value of the quality option."

When all bonds have similar net bases, the quality option is worth little (none is clearly favored). When one bond has net basis near zero while others have large positive net bases, CTD is stable and the quality option has significant value.

**Optionality behavior:** For a bond close to CTD, Tuckman notes that "the net basis of a bond close to CTD behaves like a straddle on rates or prices." Rate moves in either direction push it away from CTD and increase its net basis.

### 23.9.3 Timing Option: Early vs. Late Delivery

Tuckman describes the trade-off governing the timing option: "Under the early delivery strategy, the trader pays carry on the CTD and sacrifices any value left in the quality option. Under the late delivery strategy, the trader pays no carry and can switch bonds if the CTD changes."

The conclusion: "Clearly, if carry is positive, it is optimal to delay delivery. If carry is negative, however, then the carry advantage of delivering early must be weighed against the sacrifice of the quality option."

**Decision framework for delivery timing:**

| Carry | Quality Option Value | Optimal Strategy |
|-------|---------------------|------------------|
| Positive | Any | Delay delivery (capture carry) |
| Negative | High | Delay delivery (preserve option) |
| Negative | Low | Consider early delivery (avoid negative carry) |

> **Analogy: Rental Car Return**
>
> Think of the timing option like returning a rental car:
> - **Return early:** You pay the early return fee (give up remaining option value), but you stop paying the daily rental (negative carry stops accruing)
> - **Return late:** You keep the car until the last minute, preserving flexibility to use it if needed, but you keep paying the daily rate
>
> The optimal choice depends on how much the car is costing you (carry) versus how much the flexibility is worth (option value).

### 23.9.4 End-of-Month Option

After the last trade date, the settlement price is fixed but delivery can continue. If prices move favorably, the short can switch bonds.

Tuckman explains the P&L from switching (equation 20.17): If the CTD changes after the final settlement price $\bar{F}$ is set, the short can "sell the holding of bond $i$, buy the new CTD, and deliver the new CTD instead of bond $i$. This option to switch bonds after the last trading date is called the end-of-month option."

Tuckman: "Before the last trade date the futures price reflects any cheapening of the CTD. After the last trade date, however,... any cheapening of the CTD leads to greater and greater profits."

However, Tuckman notes the end-of-month option "does not turn out to be worth much in practice" because the time window is short and "traders long bonds and short futures actively seek opportunities to profit by switching bond holdings. This attention tends to dominate trading of bonds in the deliverable basket."

### 23.9.5 Wild Card Play

Hull describes the mechanism: trading continues after the 2:00 p.m. settlement. "If bond prices decline after 2:00 p.m. on the first day of the delivery month, the party with the short position can issue a notice of intention to deliver at, say, 3:45 p.m. and proceed to buy bonds in the spot market for delivery at a price calculated from the 2:00 p.m. futures price."

"As with the other options open to the party with the short position, the wild card play is not free. Its value is reflected in the futures price, which is lower than it would be without the option."

> **Deep Dive: The Wild Card Arbitrage**
>
> *   **2:00 PM**: Futures market closes. Settlement price $F$ is fixed.
> *   **4:00 PM**: Treasury bond market is still open.
> *   **Scenario**: Bond prices crash at 3:00 PM.
> *   **The Play**:
>     1.  Buy the physical bonds at the new, crashed price (Cheap).
>     2.  Notify the exchange you will deliver.
>     3.  Get paid based on the 2:00 PM Settlement Price (High).
> *   **Result**: Instant risk-free profit. Because this *can* happen, the futures price trades slightly lower all day long to compensate the long holder for this risk.

---

## 23.10 Futures DV01 and Hedging

### 23.10.1 Duration-Based Hedge Ratio from Hull

Hull provides the standard formula for duration-based hedging with futures (equation 6.3). Define:
- $V_F$: Contract price for one interest rate futures contract
- $D_F$: Duration of the asset underlying the futures contract
- $P$: Value of the portfolio being hedged
- $D_P$: Duration of the portfolio

The number of contracts required is:

$$\boxed{N^* = \frac{P \times D_P}{V_F \times D_F}}$$

Hull notes: "When the hedging instrument is a Treasury bond futures contract, the hedger must base $D_F$ on an assumption that one particular bond will be delivered. This means that the hedger must estimate which of the available bonds is likely to be cheapest to deliver at the time the hedge is put in place."

### 23.10.2 Futures DV01 Approximation

Near delivery, the futures price approximates:

$$F \approx \frac{P^{\text{CTD}}}{cf^{\text{CTD}}}$$

Differentiating with respect to yield:

$$\frac{dF}{dy} \approx \frac{1}{cf^{\text{CTD}}} \frac{dP^{\text{CTD}}}{dy}$$

Therefore:

$$\boxed{\text{DV01}_{\text{fut, per \$100}} \approx \frac{\text{DV01}_{\text{CTD, per \$100}}}{cf^{\text{CTD}}}}$$

For a contract with face $N = \$100{,}000$:

$$\boxed{\text{DV01}_{\text{fut, per contract}} \approx \frac{N}{100} \times \frac{\text{DV01}_{\text{CTD, per \$100}}}{cf^{\text{CTD}}}}$$

### 23.10.3 Hedge Ratio with DV01

The standard DV01 hedge ratio (from Chapter 15) is:

$$\boxed{n = \frac{\text{DV01}_{\text{exposure}}}{\text{DV01}_{\text{hedge}}}}$$

For hedging a bond position with futures, the conversion factor enters through the futures DV01.

### 23.10.4 CTD Switching Risk

**This is critical for practitioners:** If CTD switches, the futures DV01 changes abruptly. A hedge that was DV01-neutral becomes mismatched.

Hull explicitly warns: "If, subsequently, the interest rate environment changes so that it looks as though a different bond will be cheapest to deliver, then the hedge has to be adjusted and as a result its performance may be worse than anticipated."

**Example:** If CTD switches from a 7-year duration bond (CF = 0.90) to a 9-year duration bond (CF = 0.84), the futures DV01 per contract jumps significantly. A hedger who was short 450 contracts may now need only 377—or vice versa, creating instant P&L and requiring rebalancing.

Practitioners must monitor CTD status and recognize that futures hedges have embedded optionality.

### 23.10.5 Multi-Factor Considerations and Model Selection

Tuckman notes that a one-factor approach to futures hedging has limitations: "Hedging a futures contract with cash bonds alone is, at least in part, a hedge of repo rates with bonds in the delivery basket, for example, a hedge of a three-month rate with 10-year bonds."

For more sophisticated risk management, Tuckman suggests computing "both the change in futures price for a parallel shift in spot yields and the change in futures price for a parallel shift in repo rates" and hedging each exposure separately.

**The model selection problem:**

| Approach | Advantages | Disadvantages | When to Use |
|----------|------------|---------------|-------------|
| **One-factor (parallel shift)** | Simple, closed-form | Ignores repo risk, curve shape | Quick hedge ratios, stable CTD |
| **Two-factor (level + slope)** | Captures curve risk | More complex, requires calibration | CTD near switching, curve trades |
| **Basket-level model** | Full richness of each bond | Computationally intensive | Detailed basis trading, options pricing |
| **One-factor + repo sensitivity** | Separates spot/repo exposures | Requires two hedge instruments | Financed positions, carry trades |

Tuckman observes: "The discussion in the previous paragraph suggests that there is a family of one-factor measures of price sensitivity for futures contracts. It may be assumed that for every one-basis point move in the spot yield the repo rate moves by $.25, .5$, or some other fraction of a basis point. Once again, the correct choice is an empirical question."

> **Desk Reality: Choosing Your Model**
>
> For most day-to-day hedging with stable CTD, the one-factor DV01 approach is sufficient. But when CTD is close to switching—typically when multiple bonds have similar net bases—you need to think about:
> 1. What happens if CTD switches (recalculate hedge ratio under alternative CTD)
> 2. Whether your position is long or short the embedded quality option
> 3. Whether repo moves differently than spot yields (especially in stressed markets)
>
> The warning sign that you need a more sophisticated approach: unexplained P&L when the curve moved but your hedge "should have" worked.

---

## 23.11 Futures Rolls and Calendar Spreads

### 23.11.1 What Is a Roll?

A **calendar spread** (or **roll**) is the price difference between two futures contracts on the same underlying with different expiration months—typically the front month and the next (deferred) month.

$$\text{Roll} = F_{\text{front}} - F_{\text{deferred}}$$

The roll is also called the **calendar spread** because it represents the cost of extending a futures position from one delivery month to the next.

> **Practitioner Note:** Roll mechanics are not covered in detail by Tuckman or Hull, but they derive directly from the basis and carry concepts developed above. The roll is essential for practitioners because most hedgers do not intend to take delivery—they must roll their positions forward as contracts approach delivery.

### 23.11.2 Why the Roll Exists: Carry Differentials

The roll reflects the **difference in carry** between the two delivery periods. Consider:

- **Front-month futures** prices based on carry to front delivery date
- **Deferred futures** prices based on carry to back delivery date

If carry is positive (coupon income exceeds financing cost), the deferred contract will be lower in price than the front, so the roll is positive (front > deferred). If carry is negative, the roll inverts.

More precisely, the roll approximates:

$$\boxed{\text{Roll} \approx \frac{\text{Carry from front to back delivery}}{cf^{\text{CTD}}}}$$

This relationship holds when CTD is the same across both contracts. If CTD differs, the roll also reflects the relative pricing of the two CTD bonds.

### 23.11.3 Rolling the Hedge

Hedgers with positions extending beyond the front delivery month must **roll** their futures hedge. This involves:

1. **Close the front-month position** (buy back if short, sell if long)
2. **Open the deferred-month position** (sell if hedging, buy if originally short)

The net cost of rolling equals the roll spread at execution. If roll = 0.50 (front above deferred) and you're rolling a short hedge:
- You buy back front at 108.00
- You sell back at 107.50
- Cost per contract: $0.50 × 1000 = $500

> **Desk Reality: Roll Mechanics**
>
> **When do rolls trade?** The most liquid roll window is typically 2-3 weeks before first delivery date. Liquidity in the roll market is usually better than trading the two legs outright.
>
> **Roll bid-ask:** The roll market often has tighter spreads than the sum of the two outright markets because market makers can offset risk across the two contracts.
>
> **P&L attribution:** When your hedged position shows P&L during roll periods, some of it may be roll slippage—the difference between where you expected to roll and where you actually executed. This is real P&L, not noise.

### 23.11.4 When Rolls "Blow Out"

Rolls can deviate significantly from theoretical levels when:

1. **CTD differs between contracts:** If the front contract has a different CTD than the back (due to basket composition or yield changes), the roll includes both carry differential and relative value between the two CTD bonds.

2. **Squeeze in front month:** If the front-month CTD becomes extremely special or scarce (see Section 23.12), the front contract may trade rich, widening the roll.

3. **Quarter-end effects:** Balance sheet constraints on dealers can distort repo rates and roll pricing near quarter-ends.

4. **Delivery uncertainty:** Near delivery, if there's uncertainty about CTD, the front contract may trade with additional premium or discount.

### 23.11.5 Roll P&L Attribution

For a hedged position that rolls, total P&L decomposes into:

$$\text{Total P\&L} = \text{Cash P\&L} + \text{Futures MTM} + \text{Roll Slippage}$$

where:
- **Cash P&L:** Change in bond value plus carry earned
- **Futures MTM:** Change in futures positions (mark-to-market)
- **Roll Slippage:** Difference between theoretical and executed roll

Understanding this decomposition helps explain why a "hedged" position still shows P&L.

---

## 23.12 Squeeze Risk and Extreme Specialness

### 23.12.1 What Is a Squeeze?

A **squeeze** occurs when the CTD bond becomes scarce—either because it's been accumulated by investors who won't lend it, or because total deliverable supply is small relative to open interest. In a squeeze:

1. **The CTD goes extremely special** in repo (financing cost drops dramatically)
2. **Shorts scramble to locate the bond** for delivery
3. **Basis blows out** as the cash bond trades rich to futures
4. **Roll spreads distort** as the pressure concentrates in the near-delivery contract

Tuckman explains why the delivery basket exists: to prevent squeezes. A single deliverable would allow a trader to "profit by simultaneously purchasing a large fraction of that bond issue and a large number of contracts," forcing shorts to pay distorted prices.

### 23.12.2 Mechanics of Squeeze P&L

When the CTD goes from normal repo to extremely special:

1. **Carry improves dramatically** (lower financing cost)
2. **Forward price drops** (lower repo → lower forward price)
3. **Net basis may collapse toward zero** or even go negative
4. **Basis traders who are long basis get squeezed** (their hedge underperforms)

The squeeze benefits:
- **Holders of the physical CTD** (they earn special repo spread)
- **Longs in the futures** (futures catch up toward the rich cash price)

The squeeze hurts:
- **Shorts who need to deliver** (they pay up for the bond)
- **Basis traders long the basis** (if they're financing at GC while the CTD goes special)

### 23.12.3 Warning Signs of Developing Squeeze

| Indicator | What to Watch |
|-----------|---------------|
| **Declining float** | CTD issue outstanding minus Fed holdings minus long-term holders |
| **Rising special spread** | CTD financing rate well below GC |
| **Fails increasing** | Fails to deliver the CTD rising |
| **Open interest vs. deliverable** | OI approaching or exceeding deliverable supply |
| **Net basis collapsing** | CTD net basis approaching zero from above |

> **Practitioner Note:** In squeeze-like conditions, the CTD can trade extremely special (repo far below GC), making basis trade economics dramatically different from normal carry-adjusted expectations. The mechanics described here apply to any such event, regardless of the specific episode.

### 23.12.4 Operational Responses to Squeeze Risk

1. **Monitor deliverable supply** relative to open interest
2. **Consider alternative deliverables** if near-CTD bonds are available
3. **Adjust hedge ratios** if CTD switch becomes likely
4. **Use fails market** as last resort (delivering fails may be cheaper than paying up)
5. **Consider rolling early** to avoid squeeze premium in delivery month

---

## 23.13 Trading Case Study: The TYM0 Basis Trade

Tuckman provides an excellent case study of the November '08 basis trade into the June 2000 ten-year note futures contract (TYM0). This case illustrates how basis trades work, what can go wrong, and how option protection can mitigate losses.

### 23.13.1 Trade Setup (February 28, 2000)

On February 28, 2000, TYM0 appeared cheap in most dealer models. The 4.75s of November 15, 2008, had a net basis of 7.5 ticks. Traders sold the November '08 net basis—meaning they bought the bond, financed it in repo, and sold futures (with proper tailing).

Per Tuckman's Table 20.6, the deliverable basket had four bonds with maturities spanning 2007-2010. The CTD at trade initiation was the 4.75s of November 2008.

### 23.13.2 The Trade Rationale

The trade made sense if the futures was truly cheap relative to cash. The scenario analysis (Tuckman Table 20.7) showed:

- **If yields rose 40-80bp:** Net basis would fall toward 0.9 ticks. P&L from basis position: +$200k on $100mm face.
- **If yields fell 60-80bp:** Net basis would rise to 7-13 ticks. P&L from basis position: -$180k.

The asymmetry worried traders: unlimited losses in a rally, capped gains in a sell-off (once CTD switched, basis P&L stabilized).

### 23.13.3 Option Protection

Many traders bought call options on futures to hedge the rally scenario. Per Tuckman, buying 47 contracts of 95-strike calls cost 1.516 per contract (1-16.5 in ticks). This option protection:

- **Limited losses** in a rally (options gain value, offsetting basis losses)
- **Reduced gains** in a sell-off (option premium is lost)
- **Created a more symmetric P&L profile**

### 23.13.4 What Actually Happened

From February to April 2000:
- Yields rallied 47bp (April 3)
- CTD shifted toward shorter bonds (August '07, February '08)
- November '08 net basis rose to 11.06 ticks
- Basis position lost $112,813; options gained $87,391; net loss only $25,422

Then the trade reversed:
- By May 19, yields had backed up (returned close to starting levels)
- November '08 returned to near-CTD status with net basis at 3.51 ticks
- Final P&L on basis: +$123,125; options expired near worthless (-$57,281)
- **Total profit: $65,844**

### 23.13.5 Key Lessons from the Case

1. **Basis trades can work even with intermediate mark-to-market losses** — patience and proper hedge sizing matter
2. **Option protection has value** — the cost of options may be worth the reduced volatility
3. **CTD switches create P&L swings** — the trade was underwater when CTD shifted away from November '08
4. **Model-driven trades require conviction** — holding through April required belief in the model's assessment that futures was cheap
5. **Tail management matters** — Tuckman notes the proper tailing of the futures position is critical to realizing the P&L

> **Desk Reality: Basis Trading Today**
>
> The TYM0 case study illustrates mechanics that still apply. The specific numbers are dated, but the principles are timeless: basis trades are essentially bets on option value converging to zero while earning carry. When CTD is stable, you earn carry-adjusted net basis. When CTD switches, you're long or short the quality option, which can dominate the P&L.

---

## 23.14 Worked Examples

### Conventions for Examples

Unless otherwise stated:
- Prices are per $100 face
- Repo uses simple interest with ACT/360
- Futures contract size $N = \$100{,}000$ (so $N/100 = 1000$)
- Invoice price formula: $\text{Invoice} = cf \times F + AI$

### Example A: Invoice Price Calculation

**Inputs:**
- Futures settlement: $F = 112.50$
- Conversion factor: $cf = 0.9012$
- Accrued at delivery: $AI(T) = 1.35$

**Calculation:**

Delivery price: $cf \times F = 0.9012 \times 112.50 = 101.385$

Invoice price: $101.385 + 1.35 = 102.735$ per $100 face

Per contract: $102.735 \times 1000 = \$102{,}735$

### Example B: Delivery Economics

Buy bond today, finance in repo, deliver at $T$.

**Inputs:**
- Bond clean price: $P(0) = 101.20$
- Accrued today: $AI(0) = 0.45$
- Repo rate: $r = 5.00\%$
- Days to delivery: $d = 90$
- Invoice at delivery: $\text{Invoice}(T) = 102.735$ (from Example A)

**Financing repayment:**

$$101.65 \times \left(1 + \frac{0.05 \times 90}{360}\right) = 101.65 \times 1.0125 = 102.921$$

**Delivery profit:**

$$\Pi = 102.735 - 102.921 = -0.186 \text{ per \$100}$$

Per contract: $-\$186$

The trade loses money—this bond is not cheap enough at current financing rates.

### Example C: CTD Selection

**Inputs:** $F = 112.50$

| Bond | $P$ | $cf$ | $cf \times F$ | CostDel |
|------|-----|------|---------------|---------|
| A | 101.20 | 0.9012 | 101.385 | -0.185 |
| B | 109.80 | 0.9775 | 109.969 | -0.169 |
| C | 94.60 | 0.8420 | 94.725 | -0.125 |

CTD is **Bond A** (most negative cost = largest delivery profit for short).

### Example D: CTD Switch After Yield Rise

Assume yields rise 50 bp. Using DV01 approximation: $\Delta P \approx -\text{DV01} \times 50$

**DV01s:** A = 0.085, B = 0.075, C = 0.095

**New prices:**
- $P'_A = 101.20 - 4.25 = 96.95$
- $P'_B = 109.80 - 3.75 = 106.05$
- $P'_C = 94.60 - 4.75 = 89.85$

**New futures:** $F' = 107.00$

**New costs:**
- $\text{CostDel}'_A = 96.95 - 0.9012 \times 107 = 96.95 - 96.43 = +0.52$
- $\text{CostDel}'_B = 106.05 - 0.9775 \times 107 = 106.05 - 104.59 = +1.46$
- $\text{CostDel}'_C = 89.85 - 0.8420 \times 107 = 89.85 - 90.09 = -0.24$

CTD switches from **A to C**. The higher-duration bond becomes CTD when yields rise.

### Example E: Gross and Net Basis

**Inputs for Bond A:**
- $P(0) = 101.20$, $AI(0) = 0.45$, $AI(T) = 1.35$
- $cf = 0.9012$, $F = 112.50$
- Repo rate: 5.00%, $d = 90$ days

**Gross basis:**
$$GB = 101.20 - 0.9012 \times 112.50 = 101.20 - 101.385 = -0.185$$

**Carry:**
Interest income: $AI(T) - AI(0) = 1.35 - 0.45 = 0.90$

Financing cost: $101.65 \times 0.05 \times 90/360 = 1.271$

$$\text{Carry} = 0.90 - 1.271 = -0.371$$

**Net basis:**
$$NB = GB - \text{Carry} = -0.185 - (-0.371) = +0.186$$

Positive net basis means cash-and-carry loses money at this financing rate (consistent with Example B).

### Example F: Implied Repo Rate

**Inputs:**
- Dirty price today: $101.65$
- Invoice at delivery: $102.735$
- Days: $d = 90$

**Break-even condition:**
$$101.65(1 + r_{\text{imp}} \times 90/360) = 102.735$$

$$r_{\text{imp}} = \left(\frac{102.735}{101.65} - 1\right) \times \frac{360}{90} = 0.01067 \times 4 = 4.27\%$$

If you can finance below 4.27%, cash-and-carry is profitable. At 5% GC, it loses money.

### Example G: Futures DV01 and Hedge Ratio

**Inputs:**
- Cash position: long $50 million face, DV01 = 0.085 per $100
- CTD: Bond A with DV01 = 0.085, cf = 0.9012

**Cash DV01 in dollars:**
$$0.085 \times \frac{50{,}000{,}000}{100} = \$42{,}500 \text{ per bp}$$

**Futures DV01 per contract:**
$$1000 \times \frac{0.085}{0.9012} = 1000 \times 0.0943 = \$94.32 \text{ per bp}$$

**Hedge ratio:**
$$n = \frac{42{,}500}{94.32} \approx 451 \text{ contracts}$$

**If CTD switches to C** (DV01 = 0.095, cf = 0.8420):

$$\text{Futures DV01} = 1000 \times \frac{0.095}{0.8420} = \$112.83 \text{ per bp}$$

$$n' = \frac{42{,}500}{112.83} \approx 377 \text{ contracts}$$

The hedge ratio changes by **74 contracts** purely from CTD switching.

### Example H: Roll Calculation

**Inputs:**
- Front-month futures (Mar): $F_{\text{Mar}} = 112.50$
- Back-month futures (Jun): $F_{\text{Jun}} = 112.15$
- CTD same for both contracts

**Roll:**
$$\text{Roll} = F_{\text{Mar}} - F_{\text{Jun}} = 112.50 - 112.15 = 0.35$$

**Cost of rolling 100 contracts (short hedge):**
- Buy back Mar at 112.50
- Sell Jun at 112.15
- Net cost per contract: $0.35 \times 1000 = \$350$
- Total roll cost: $100 \times \$350 = \$35,000$

**Sanity check:** The roll should approximate carry from Mar to Jun delivery. If carry is approximately $\$0.35$ per $100 over 3 months, this is consistent.

---

## 23.15 Practical Notes

### Contract-Specific Checklist

| Item | What to Verify |
|------|----------------|
| Deliverable basket rules | Maturity range, eligible issues, exclusions |
| Conversion factor formula | 6% discounting, rounding rules (quarter vs month) |
| Delivery window | First delivery date, last trade date, last delivery date |
| Invoice rounding | Minimum tick, accrued interest conventions |
| Settlement timing | Mark-to-market timing, delivery notice cutoffs |

### Key Contract Parameters (Approximate)

| Contract | Maturity Range | Typical CTD Duration | Tick Size |
|----------|---------------|---------------------|-----------|
| 2-Year (TU) | 1.75 - 2 years | ~1.8 years | 1/128 |
| 5-Year (FV) | 4.2 - 5.25 years | ~4.5 years | 1/128 |
| 10-Year (TY) | 6.5 - 10 years | ~7-8 years | 1/64 |
| T-Bond (US) | 15+ years | ~15-20 years | 1/32 |
| Ultra T-Bond (UB) | 25+ years | ~25 years | 1/32 |

*These are approximate parameters for intuition. Exchange specifications evolve; always verify current rules (eligible deliverables, ticks, timing) before trading.*

### Common Pitfalls

1. **Mixing clean and dirty prices:** Invoice price adds accrued; cost of delivery uses clean
2. **Treating futures DV01 as stable:** CTD can switch, causing sudden hedge mismatch
3. **Ignoring repo specialness:** GC vs special materially affects carry and CTD economics
4. **Confusing gross vs net basis:** Net basis (carry-adjusted) is economically relevant for basis trades
5. **Assuming CTD is constant:** Changes in yield level or curve shape can shift CTD, requiring hedge adjustment
6. **Forgetting the roll:** Hedges must be rolled before delivery; roll slippage is real P&L

### Sanity Checks

- **CTD reproducibility:** Compute $P - cf \times F$ for all deliverables; CTD must be the minimum
- **Implied repo vs market:** Extreme implied repo (very high or negative) suggests checking accrued/coupon assumptions
- **Hedge ratio stability:** Monitor CTD status; rebalance when CTD is close to switching
- **Roll reasonableness:** Roll should approximately equal carry between delivery months

---

## Summary

Treasury futures are **deliverable-basket contracts** where the short holds embedded options—most importantly, the quality option to choose which bond to deliver and timing options about when to deliver. These options transfer value from long to short and depress the futures price below simple cost-of-carry levels.

**Key takeaways:**

1. **Conversion factors** standardize delivery across bonds of different coupons and maturities, computed by discounting at a notional 6% yield

2. **CTD minimizes cost of delivery:** $\text{CTD} = \arg\min_i\{P_i - cf_i \times F\}$

3. **CTD switches** with yield level (high yields favor high duration) and curve shape—this creates risk for hedgers

4. **Negative convexity:** The futures contract exhibits negative convexity because the short always delivers the bond that minimizes value—duration falls as yields fall

5. **Net basis = gross basis minus carry** and represents the quality option value that can be locked in

6. **Implied repo rate** measures whether cash-and-carry is attractive relative to funding costs

7. **Repo specialness** affects carry and can shift CTD economics

8. **Futures DV01 ≈ CTD DV01 / cf** but jumps when CTD switches—hedge ratios are not stable

9. **Rolls** are the cost of extending a futures position and derive from carry differentials between delivery months

10. **Squeeze risk** materializes when CTD becomes scarce, causing basis to blow out

11. **Model selection** depends on CTD stability: one-factor for stable CTD, multi-factor when CTD is near switching or for detailed basis trading

12. Hull warns: exact pricing is "difficult" because the short's delivery options "cannot easily be valued"

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| Conversion factor | Bond price at 6% yield per $1 face; scales invoice price | Links futures price to individual bond delivery |
| Cost of delivery | $P - cf \times F$; bond cost minus invoice | Determines which bond is CTD |
| CTD (Cheapest-to-deliver) | Bond minimizing cost of delivery | Central to pricing, hedging, and basis trading |
| Gross basis | $P - cf \times F$ (same as cost of delivery) | Measures cash-futures price relationship |
| Net basis | $P_{\text{fwd}} - cf \times F = GB - \text{Carry}$ | Carry-adjusted measure; proxy for option value |
| Implied repo rate | Break-even financing rate for cash-and-carry | Compares trade economics to actual funding |
| Quality option | Short's right to choose delivery bond | Creates negative convexity in futures |
| Timing option | Short's right to choose delivery date | Additional optionality for short |
| Negative convexity | Duration falls as yields fall due to CTD switching | Futures don't behave like simple bonds |
| Calendar spread (roll) | Front minus deferred futures price | Cost of extending futures position |
| Squeeze | Scarcity of CTD causing basis blow-out | Extreme risk scenario for basis traders |

---

## Notation for This Chapter

| Symbol | Definition |
|--------|------------|
| $F(t)$ | Futures settlement price at time $t$ (per $100 notional) |
| $cf_i$ | Conversion factor for bond $i$ (dimensionless) |
| $P_i(t)$ | Clean price of bond $i$ at time $t$ (per $100 face) |
| $AI_i(t)$ | Accrued interest on bond $i$ at time $t$ (per $100 face) |
| $\text{Invoice}_i(t)$ | Cash received on delivery of bond $i$: $cf_i \times F(t) + AI_i(t)$ |
| $\text{CostDel}_i(t)$ | Cost of delivery: $P_i(t) - cf_i \times F(t)$ |
| $GB^i(t)$ | Gross basis (= cost of delivery) |
| $NB^i(t)$ | Net basis: $P_{\text{fwd}}^i(t) - cf_i \times F(t)$ |
| $r$ | Repo rate (annualized, ACT/360) |
| $d$ | Days to delivery |
| $N$ | Contract face amount ($100,000 typically) |
| $D_F$ | Duration of asset underlying futures (CTD) |
| $D_P$ | Duration of portfolio being hedged |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is the "delivery basket" in Treasury futures? | The set of eligible Treasury securities the short may choose to deliver |
| 2 | Who controls which bond is delivered? | The short position holder (quality option) |
| 3 | How is the conversion factor calculated? | As the bond's price per $1 face when discounted at 6% semiannual |
| 4 | What is the invoice price formula? | $cf_i \times F(t) + AI_i(t)$ |
| 5 | Why is accrued interest added to delivery price? | Because bonds exchange at dirty (cash) price |
| 6 | Write the cost of delivery formula | $\text{CostDel}_i = P_i - cf_i \times F$ |
| 7 | Why does AI cancel in cost of delivery? | It appears in both purchase cost and invoice received |
| 8 | Define CTD | The bond minimizing cost of delivery (maximizing delivery profit) |
| 9 | What is the futures price at delivery per Tuckman? | $F(T) = P^{CTD}(T) / cf^{CTD}$ |
| 10 | What does gross basis measure? | Cash price minus CF-adjusted futures price |
| 11 | What does net basis measure? | Forward price minus CF-adjusted futures price (carry-adjusted) |
| 12 | How is net basis related to gross basis? | $NB = GB - \text{Carry}$ |
| 13 | What does Tuckman interpret net basis as? | The value of the quality option |
| 14 | When is net basis near zero? | When quality option is nearly worthless |
| 15 | Give the forward price formula under repo | $P_{\text{fwd}} + AI(T) = (P(0) + AI(0))(1 + rd/360)$ |
| 16 | What is the timing option? | Short's right to choose delivery date within the window |
| 17 | What is the wild card play? | Option from post-settlement trading allowing late delivery notice |
| 18 | What is the end-of-month option? | Option to switch bonds after last trade but before last delivery |
| 19 | Why are futures prices "difficult" to determine exactly (per Hull)? | Delivery options cannot easily be valued |
| 20 | Which bonds tend to be CTD when yields are high (above 6%)? | Long duration (low coupon, long maturity) |
| 21 | Which bonds tend to be CTD when yields are low (below 6%)? | Short duration (high coupon, short maturity) |
| 22 | Why are Treasury futures negatively convex? | CTD switching: duration falls as yields fall |
| 23 | What is implied repo rate? | Financing rate at which cash-and-carry breaks even |
| 24 | If implied repo > actual repo, is cash-and-carry attractive? | Yes—trade earns more than financing costs |
| 25 | What is the P&L formula for a long basis trade? | $G^i \times [NB^i(t') - NB^i(t)]$ |
| 26 | Give the futures DV01 approximation | $\text{DV01}_{\text{fut}} \approx \text{DV01}_{\text{CTD}}/cf_{\text{CTD}}$ |
| 27 | What is CTD switching risk? | Risk that CTD changes, causing hedge ratio to jump |
| 28 | What is Hull's duration-based hedge ratio formula? | $N^* = (P \times D_P) / (V_F \times D_F)$ |
| 29 | What must a hedger estimate when using Treasury futures per Hull? | Which bond is likely to be cheapest to deliver |
| 30 | What is a calendar spread (roll)? | Price difference between front and deferred futures |
| 31 | What drives roll pricing? | Carry differential between delivery months |
| 32 | What is a squeeze in Treasury futures? | Scarcity of CTD causing extreme specialness and basis blow-out |
| 33 | Why does the delivery basket exist? | To prevent squeezes and maintain contract liquidity |
| 34 | When should you use a multi-factor model for futures? | When CTD is close to switching or for detailed basis trading |
| 35 | What is the key practitioner message? | Treasury futures = delivery optionality + financing (repo) |

---

## Mini Problem Set

### Questions 1–8 (With Solution Sketches)

**1.** Compute invoice price given $F = 108.50$, $cf = 0.8750$, $AI = 2.10$.

*Sketch:* Invoice $= 0.8750 \times 108.50 + 2.10 = 94.94 + 2.10 = 97.04$ per $100. Per contract: $\$97{,}040$.

**2.** Given three deliverables with $(P, cf) = [(98.50, 0.92), (105.00, 0.98), (91.20, 0.85)]$ and $F = 106.00$, identify CTD.

*Sketch:* Compute $P - cf \times F$: $(98.50 - 97.52) = 0.98$; $(105 - 103.88) = 1.12$; $(91.20 - 90.10) = 1.10$. CTD is Bond 1 (lowest = 0.98).

**3.** Compute gross basis for Bond A: $P = 102.50$, $cf = 0.9500$, $F = 108.00$.

*Sketch:* $GB = 102.50 - 0.95 \times 108 = 102.50 - 102.60 = -0.10$. Negative = cash cheap vs futures.

**4.** Compute carry for 60 days: $P + AI(0) = 103.20$, $AI(T) - AI(0) = 1.00$, $r = 4.50\%$.

*Sketch:* Financing = $103.20 \times 0.045 \times 60/360 = 0.774$. Carry $= 1.00 - 0.774 = 0.226$.

**5.** Compute net basis from gross basis $= -0.10$ and carry $= 0.226$.

*Sketch:* $NB = GB - \text{Carry} = -0.10 - 0.226 = -0.326$.

**6.** Compute implied repo: dirty today $= 103.20$, invoice at delivery $= 103.85$, $d = 60$ days.

*Sketch:* $r_{\text{imp}} = (103.85/103.20 - 1) \times 360/60 = 0.0063 \times 6 = 3.78\%$.

**7.** If GC = 4.50% and special = 3.00%, what is the special spread?

*Sketch:* Special spread $= 4.50\% - 3.00\% = 150$ bp.

**8.** Cash DV01 = $\$30{,}000$/bp. CTD has DV01 = 0.072 per $100, cf = 0.8500. Compute hedge ratio.

*Sketch:* Futures DV01 per contract $= 1000 \times 0.072/0.85 = \$84.71$/bp. Hedge $= 30{,}000/84.71 \approx 354$ contracts.

### Questions 9–18 (No Solutions Provided)

**9.** Explain qualitatively why the futures price is not a pure cost-of-carry forward price.

**10.** Describe how a CTD switch can create P&L on an otherwise DV01-neutral hedge.

**11.** Construct a scenario where gross basis is negative but net basis is positive. Why might a basis trade lose money?

**12.** A bond becomes extremely special in repo. Explain how it might become CTD even if its cash price rises.

**13.** What market feature creates the wild card play (per Hull), and how does it benefit the short?

**14.** Tuckman notes the end-of-month option "does not turn out to be worth much in practice." Explain why.

**15.** Explain net basis as quality option value using Tuckman's interpretation.

**16.** The roll spread for Mar/Jun widens from 0.35 to 0.60. List three possible causes.

**17.** Design a daily monitoring checklist for a live Treasury futures hedge.

**18.** A squeeze develops in the front-month CTD. Describe the impact on (a) basis trades, (b) roll spreads, (c) hedgers with delivery approaching.

---

## References

- Tuckman & Serrat, *Fixed Income Securities: Tools for Today’s Markets* (Treasury futures CTD, basis, implied repo, and delivery options; case studies).
- Hull, *Options, Futures, and Other Derivatives* (Treasury futures hedging, conversion factors, and delivery-option intuition).
- CBOT/CME historical documentation on the 8% → 6% notional coupon change (approved Feb 1999; effective Mar 2000).
- CME contract specifications / rulebook (current deliverable-basket and tick/settlement details).
