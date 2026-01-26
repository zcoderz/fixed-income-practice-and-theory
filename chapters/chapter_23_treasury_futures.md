# Chapter 23: Treasury Futures — CTD, Conversion Factors, Delivery Options, and the Link to Repo

---

## Introduction

The Treasury bond futures contract doesn't specify which bond you deliver—so which one do you choose?

This seemingly simple question reveals the distinctive feature that makes bond futures pricing both fascinating and challenging. Unlike most futures contracts where the underlying asset is precisely defined, Treasury futures allow the short position to deliver any bond from a specified basket. The short can also choose *when* to deliver within an extended window. These embedded options—the **quality option** to choose the bond and the **timing option** to choose the date—transfer value from the long to the short and depress the futures price below what a simple cost-of-carry calculation would suggest.

Understanding this mechanism is essential for anyone trading, hedging, or pricing with Treasury futures. A hedger who ignores the delivery option dynamics may find their carefully constructed DV01 hedge suddenly misaligned when the cheapest-to-deliver bond switches. A basis trader who focuses only on gross basis without accounting for carry and option value will systematically misprice the trade's expected return. As Hull notes explicitly: "An exact theoretical futures price for the Treasury bond contract is difficult to determine because the short party's options concerned with the timing of delivery and choice of the bond that is delivered cannot easily be valued."

This chapter covers:

1. **The deliverable basket and conversion factors** — how the contract standardizes delivery across bonds of different coupons and maturities
2. **Cost of delivery and the CTD** — how the short identifies the optimal bond to deliver
3. **When CTD switches** — how yield levels, curve shape, and financing conditions shift CTD
4. **Gross basis, net basis, and implied repo** — the metrics practitioners use for cash-futures relative value
5. **Delivery options and their value** — quality, timing, end-of-month, and wild card features
6. **The repo-futures link** — how financing costs enter through carry and specialness
7. **Hedging implications** — why futures DV01 depends on CTD and can jump

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

### 23.9.3 Timing Option

Tuckman describes the trade-off governing the timing option: "Under the early delivery strategy, the trader pays carry on the CTD and sacrifices any value left in the quality option. Under the late delivery strategy, the trader pays no carry and can switch bonds if the CTD changes."

The conclusion: "Clearly, if carry is positive, it is optimal to delay delivery. If carry is negative, however, then the carry advantage of delivering early must be weighed against the sacrifice of the quality option."

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

### 23.10.5 Multi-Factor Considerations

Tuckman notes that a one-factor approach to futures hedging has limitations: "Hedging a futures contract with cash bonds alone is, at least in part, a hedge of repo rates with bonds in the delivery basket, for example, a hedge of a three-month rate with 10-year bonds."

For more sophisticated risk management, Tuckman suggests computing "both the change in futures price for a parallel shift in spot yields and the change in futures price for a parallel shift in repo rates" and hedging each exposure separately.

---

## 23.11 Worked Examples

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

---

## 23.12 Practical Notes

### Contract-Specific Checklist

| Item | What to Verify |
|------|----------------|
| Deliverable basket rules | Maturity range, eligible issues, exclusions |
| Conversion factor formula | 6% discounting, rounding rules (quarter vs month) |
| Delivery window | First delivery date, last trade date, last delivery date |
| Invoice rounding | Minimum tick, accrued interest conventions |
| Settlement timing | Mark-to-market timing, delivery notice cutoffs |

*I'm not sure about the exact current specifications for specific contracts (2Y/5Y/10Y/Ultra) without the exchange rulebook version.*

### Common Pitfalls

1. **Mixing clean and dirty prices:** Invoice price adds accrued; cost of delivery uses clean
2. **Treating futures DV01 as stable:** CTD can switch, causing sudden hedge mismatch
3. **Ignoring repo specialness:** GC vs special materially affects carry and CTD economics
4. **Confusing gross vs net basis:** Net basis (carry-adjusted) is economically relevant for basis trades
5. **Assuming CTD is constant:** Changes in yield level or curve shape can shift CTD, requiring hedge adjustment

### Sanity Checks

- **CTD reproducibility:** Compute $P - cf \times F$ for all deliverables; CTD must be the minimum
- **Implied repo vs market:** Extreme implied repo (very high or negative) suggests checking accrued/coupon assumptions
- **Hedge ratio stability:** Monitor CTD status; rebalance when CTD is close to switching

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

9. Hull warns: exact pricing is "difficult" because the short's delivery options "cannot easily be valued"

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
| 30 | What is the key practitioner message? | Treasury futures = delivery optionality + financing (repo) |

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

### Questions 9–16 (No Solutions Provided)

**9.** Explain qualitatively why the futures price is not a pure cost-of-carry forward price.

**10.** Describe how a CTD switch can create P&L on an otherwise DV01-neutral hedge.

**11.** Construct a scenario where gross basis is negative but net basis is positive. Why might a basis trade lose money?

**12.** A bond becomes extremely special in repo. Explain how it might become CTD even if its cash price rises.

**13.** What market feature creates the wild card play (per Hull), and how does it benefit the short?

**14.** Tuckman notes the end-of-month option "does not turn out to be worth much in practice." Explain why.

**15.** Explain net basis as quality option value using Tuckman's interpretation.

**16.** Design a daily monitoring checklist for a live Treasury futures hedge.

---

## Source Map

### (A) Verified Facts (Source-Backed)

| Concept | Source |
|---------|--------|
| Deliverable basket, quality option, timing option definitions | Tuckman Ch 20, Hull Ch 6 |
| Invoice price: $cf \times F + AI$ | Tuckman Ch 20, Hull Ch 6 |
| Conversion factor = 6% discounting rule | Hull Ch 6 (explicit calculation examples), Tuckman Ch 20 |
| Conversion factor worked examples (10% 20yr → 1.4623; 8% 18y4m → 1.2199) | Hull Ch 6 |
| Cost of delivery: $P - cf \times F$ | Tuckman Eq 20.1 |
| CTD minimizes cost of delivery | Tuckman Ch 20, Hull Ch 6 |
| Futures price at delivery: $F(T) = P^{CTD}/cf^{CTD}$ | Tuckman Eq 20.3 |
| Arbitrage proof of futures price | Tuckman Ch 20 |
| Gross basis definition | Tuckman Eq 20.10 |
| Net basis definition | Tuckman Eq 20.11 |
| Net basis = gross basis - carry | Tuckman Eq 20.12 |
| Net basis = quality option value interpretation | Tuckman Ch 20 |
| Basis trade P&L = change in net basis × face | Tuckman Eq 20.15 |
| DV01 and hedge ratio concepts | Tuckman Ch 5-6, Hull Ch 6 |
| Duration-based hedge formula $N^* = (P \times D_P)/(V_F \times D_F)$ | Hull Eq 6.3 |
| Special spread and repo mechanics | Tuckman Ch 15-16 |
| End-of-month option definition and P&L | Tuckman Ch 20 |
| Wild card play description | Hull Business Snapshot 6.2 |
| Delivery options reduce futures price | Hull Ch 6 (explicit statement) |
| Yield level effects on CTD (high yield → high duration CTD) | Tuckman Ch 20, Hull Ch 6 |
| Curve shape effects on CTD | Tuckman Ch 20, Hull Ch 6 |
| Duration determines price/CF slope | Tuckman Ch 20 |
| Negative convexity of futures | Tuckman Ch 20 ("the contract is negatively convex") |
| Exact pricing is "difficult" due to delivery options | Hull Ch 6 |
| CTD switch requires hedge adjustment | Hull Ch 6 |

### (B) Reasoned Inference (Derived from A)

| Derived Result | Derivation Chain |
|----------------|------------------|
| Net basis formula: $NB = GB - \text{Carry}$ | Substitution of $P_{\text{fwd}} = P - \text{Carry}$ into net basis definition (Tuckman Eq 20.11-20.12) |
| Implied repo formula | Break-even condition on cash-and-carry: set delivery profit = 0 and solve for $r$ |
| Futures DV01 $\approx \text{DV01}_{\text{CTD}}/cf$ | Differentiation of $F \approx P/cf$ with respect to yield |
| Accrued cancels in cost of delivery | Algebraic: $(P + AI) - (cf \times F + AI) = P - cf \times F$ |
| Net basis behaves like straddle near CTD | Tuckman's observation that rate moves in either direction increase net basis |

### (C) Flagged Uncertainties

| Item | Uncertainty |
|------|-------------|
| Exact fails-charge formula | Sources discuss fails conceptually but don't specify penalty mechanics |
| Precise delivery notice deadlines | Contract-specific; depends on exact exchange rulebook version |
| Repo conventions outside USD | Would need additional source verification |
| Operational specialness bounds | Tuckman discusses conceptually but precise bounds are contract/market specific |
| Current standard contract specs (2Y/5Y/10Y/Ultra) | Exchange rules evolve; I'm not sure without current rulebook |
| Historical notional coupon (was 8%, now 6%) | I'm not sure exactly when the change occurred without additional verification |

---

*Last Updated: January 2026*
