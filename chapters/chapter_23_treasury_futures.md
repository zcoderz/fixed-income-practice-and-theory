# Chapter 23: Treasury Futures — CTD, Conversion Factors, Delivery Options, and the Link to Repo

---

## Introduction

The Treasury bond futures contract doesn't specify which bond you deliver—so which one do you choose?

Unlike most futures, the “underlying” is not a single bond. Treasury futures are **deliverable-basket contracts**: the short can choose *which* eligible bond to deliver and *when* (within the delivery window). These embedded delivery options transfer value from the long to the short and push the futures price away from a naive single-asset cost-of-carry forward. Exact theoretical pricing is therefore harder than for a single identified bond, but the contract is still highly structured: you can translate quotes into invoice prices, delivery economics, basis metrics, and hedge ratios.

## Learning Objectives
- After this chapter, you can describe the deliverable basket and the short’s delivery options (quality, timing, end-of-month, and “wild card” style notice timing).
- You can compute invoice price and cost of delivery, and identify the cheapest-to-deliver (CTD) bond from a deliverables table.
- You can translate cash–futures quotes into gross basis, net basis, and implied repo, and explain how funding assumptions (GC vs special) change carry and CTD economics.
- You can compute and interpret Treasury futures DV01 with an explicit bump object, bump size, units, and sign convention.
- You can explain why hedge ratios jump when CTD switches, and what “roll” and “squeeze risk” mean for live hedges.

Prerequisites: [Chapter 5 — Fixed-Rate Bond Pricing](chapter_05_fixed_rate_bond_pricing.md), [Chapter 9 — Repo Funding Engine](chapter_09_repo_funding_engine.md), [Chapter 11 — DV01/PV01 Definitions and Computation](chapter_11_dv01_pv01_definitions_computation.md), [Chapter 12 — Duration (Macaulay, Modified, DV01)](chapter_12_duration_macaulay_modified_dv01.md), [Chapter 13 — Convexity](chapter_13_convexity.md)  
Follow-on: [Chapter 24 — STIR Futures and Convexity Adjustments](chapter_24_stir_futures_convexity_adjustments.md), [Chapter 28 — Basis Trades in Rates](chapter_28_basis_trades.md)

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

This chapter leans heavily on bond pricing and accrued interest (Chapter 5), repo/carry mechanics (Chapter 9), and DV01/duration/convexity conventions (Chapters 11–13). It focuses on what makes Treasury futures *different* from forwards: the delivery basket and the short’s embedded options.

---

## 23.1 Treasury Futures as a Deliverable-Basket Contract

### 23.1.1 The Contract Structure

Treasury futures are *deliverable-basket* contracts: the short satisfies the contract by delivering **one** eligible Treasury security from a specified basket during a delivery window. The exchange specifies eligibility (maturity range and other criteria) and publishes a list of deliverables for each contract month.

Why a basket? Two practical reasons:

1. **Liquidity protection:** if one specific issue becomes illiquid or “locked up,” the futures contract can still function.
2. **Anti-squeeze design:** allowing multiple deliverables makes it harder (though not impossible) to corner a single issue and force distorted delivery economics.

In common U.S. Treasury futures, the contract face amount is $N=USD100{,}000$. The delivery cash amount is an **invoice price** based on the futures settlement price and a bond-specific **conversion factor**.

> **Key insight:** This futures is not a forward on a single bond. It is a standardized promise to deliver *some* eligible bond, and the short gets to choose.

### 23.1.2 The Short's Embedded Options

Because the short controls delivery, Treasury futures embed several short-owned options:

- **Quality option:** choose *which* eligible bond to deliver (choose the cheapest-to-deliver bond).
- **Timing option:** choose *when* to deliver within the window.
- **End-of-month option:** when the last trading day is before the last delivery day, the settlement price is fixed while delivery choices remain.
- **Wild-card style notice option:** when the delivery notice deadline occurs after the futures settlement time while the cash bond market is still trading, the short can condition the notice decision on post-settlement price moves.

All of these options are economically favorable to the short. In competitive markets, their value is reflected in the futures price.

> **Desk Reality:** Risk reports often map a Treasury futures position to “today’s CTD” and compute DV01 off that mapping.  
> **Common break:** P&L can move even when reported DV01 is near zero (CTD switching, delivery-option convexity, and repo-driven carry can dominate).  
> **What to check:** monitor net basis levels across near-CTDs, and re-run DV01 under plausible alternative CTDs before major roll/delivery periods.

---

## 23.2 Conversion Factors: Standardizing the Delivery Basket

### 23.2.1 Why Conversion Factors Exist

Bonds in the delivery basket differ in coupon and maturity. Without any adjustment, a high-coupon bond would be much more valuable than a low-coupon bond of the same maturity. The futures contract would then essentially become a contract on the highest-coupon bond, since the short would never deliver anything else.

**Conversion factors** solve this problem by translating a single futures quote into a bond-specific delivery price. They reduce (but do not eliminate) differences in delivery value across bonds with different coupons and maturities.

### 23.2.2 The 6% Rule: How Conversion Factors Are Calculated

For U.S. Treasury bond/note futures, a common definition sets a bond’s conversion factor equal to the bond’s quoted price per dollar of principal on the first day of the delivery month, assuming a flat **6%** yield curve with semiannual compounding (with contract-specific standardization/rounding of coupon timing and maturity).

In practice, you treat $cf_i$ as an exchange-published input for each eligible bond.

**Illustrative calculation (stylized, on-cycle bond):** Consider a 10% coupon bond with 20 years to maturity (40 semiannual periods). If the yield is 6% per year with semiannual compounding (3% per half-year), the theoretical clean price per USD100 is:

$$\text{Value at 6 percent} = \sum_{i=1}^{40} \frac{5}{1.03^{i}} + \frac{100}{1.03^{40}} = USD146.23$$

Dividing by the face value gives a conversion factor of **1.4623**.

**Illustrative calculation (off-cycle settlement intuition):** Consider an 8% coupon bond that is not exactly on a coupon date. One common approach is to value cashflows at 6% and then subtract accrued interest to get a clean price. For example, discounting all payments back to a point 3 months from today at 6% per year (compounded semiannually) gives:

$$4 + \sum_{i=1}^{36} \frac{4}{1.03^{i}} + \frac{100}{1.03^{36}} = USD125.8323$$

The interest rate for a 3-month period is $\sqrt{1.03} - 1 = 1.4889\\%$. Hence, discounting back to the present gives the bond's value as USD125.8323 / 1.014889 = USD123.99. Subtracting the accrued interest of 2.0 gives USD121.99. The conversion factor is therefore **1.2199**.

> **Note:** Some older references (or non-U.S. contracts) may use a different notional yield or different rounding rules. For trading and operations, always use the exchange’s published conversion factors for the contract month.

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

$$\boxed{Invoice_i(t) = cf_i \times F(t) + AI_i(t)}$$

In words: the clean delivery price is $cf_i \times F(t)$, and the invoice price adds accrued interest (delivery price + accrued interest).

where:
- $cf_i$ is the conversion factor for bond $i$
- $F(t)$ is the futures settlement price at time $t$
- $AI_i(t)$ is the accrued interest on bond $i$ at time $t$

**Intuition:** The futures price is quoted like a clean price. The invoice adds accrued interest because the delivered bond exchanges at its dirty (cash) price.

**Quick example:** Suppose the most recent settlement price is 120.00, the conversion factor for the bond delivered is 1.3800, and accrued interest at delivery is USD3 per USD100 face value. The cash received is:

$$(1.3800 \times 120.00) + 3.00 = USD168.60$$

per USD100 face value. A party with the short position in one contract would deliver bonds with face value of USD100,000 and receive USD168,600.

> **Pitfall — Invoice vs clean quote:** Mixing futures quotes, bond clean prices, and dirty cash settlement amounts.  
> **Why it matters:** You will miscompute delivery economics, implied repo, and hedge ratios by dollars per USD100 (which becomes thousands per contract).  
> **Quick check:** At delivery, cash paid/received must be “dirty”: verify the same $AI(t)$ term is added on both sides before canceling it in any spread.

### 23.3.2 Cost of Delivery

The short’s delivery economics are driven by the difference between:

- the **dirty** cost to acquire the bond: $P_i(t)+AI_i(t)$,
- the **dirty** invoice received: $cf_iF(t)+AI_i(t)$.

Accrued interest cancels, giving the clean-price expression:

$$CostDel_i(t)=\left(P_i(t)+AI_i(t)\right)-\left(cf_i \times F(t)+AI_i(t)\right)=P_i(t)-cf_i \times F(t).$$

$$\boxed{CostDel_i(t) = P_i(t) - cf_i \times F(t)}$$

**Unit check:** All terms are in dollars per USD100 face. A negative cost of delivery means delivering bond $i$ is profitable for the short at that futures price.

---

## 23.4 The Cheapest-to-Deliver (CTD) Bond

### 23.4.1 The CTD Criterion

The **cheapest-to-deliver** bond is the one that minimizes the cost of delivery—or equivalently, maximizes the short's profit from delivering.

$$\boxed{\text{CTD} = \arg\min_i \\{P_i(t) - cf_i \times F(t)\\}}$$

> **Analogy: The Fixed Price Menu**
>
> Imagine you are a short seller committed to buying dinner for the long holder.
> *   **The Contract**: You sold a "Dinner Futures" contract for USD20. You must deliver *one* item from the menu.
> *   **The Menu**:
>     *   Burger: Cost USD10. Adjusted Price = USD10 / 1.0 = USD10.
>     *   Steak: Cost USD25. Adjusted Price = USD25 / 2.0 = USD12.50.
>     *   Salad: Cost USD8. Adjusted Price = USD8 / 0.9 = USD8.88.
>
> *   **The Choice**: You will obviously buy the salad (USD8) because it's the cheapest way to fulfill the contract, even though the contract price is fixed.
> *   **The Switch**: If salad prices skyrocket to USD15, you will switch to buying the burger. This ability to *switch* is the "Quality Option." It's valuable to you (Short), and risky for the eater (Long).

**Example:** A short considers three deliverable bonds when the most recent settlement price is 93-08 (93.25):

| Bond | Quoted Price | Conversion Factor | Cost of Delivery |
|------|--------------|-------------------|------------------|
| 1 | USD99.50 | 1.0382 | USD99.50 - 93.25 × 1.0382 = USD2.69 |
| 2 | USD143.50 | 1.5188 | USD143.50 - 93.25 × 1.5188 = USD1.87 |
| 3 | USD119.75 | 1.2615 | USD119.75 - 93.25 × 1.2615 = USD2.12 |

Bond 2 has the lowest cost of delivery and is therefore CTD.

### 23.4.2 Final Settlement Price at Delivery

At the final delivery date $T$, no-arbitrage implies the settlement price must satisfy:

$$\boxed{F(T) = \min_i \frac{P_i(T)}{cf_i} = \frac{P^{\text{CTD}}(T)}{cf^{\text{CTD}}}}$$

Reason (intuition): if $F(T)$ were above the minimum $P/cf$, you could buy the cheapest bond, sell futures, deliver, and lock in a profit. If $F(T)$ were below the minimum $P/cf$, you could sell the cheapest bond, buy futures, take delivery, and lock in a profit. In frictionless markets, these arbitrages force the equality.

A key consequence: **the cost of delivering the CTD at final settlement is zero**:

$$P^{\text{CTD}}(T) - cf^{\text{CTD}} \times F(T) = 0$$

For any non-CTD bond $j$:

$$P^j(T) - cf^j \times F(T) \gt 0$$

The cost of delivering a non-CTD bond is positive.

---

## 23.5 Why CTD Can Switch: Yield Level and Curve Shape

### 23.5.1 The Imperfection of Conversion Factors

Conversion factors would equalize delivery attractiveness only in a special case (think: a flat curve at the notional yield and no rounding frictions). In reality, yields are not exactly 6% and the curve is not flat. Because $cf_i$ is fixed, bonds with different durations respond differently to yield changes, and the identity of the CTD can change.

A useful local approximation is to look at the “adjusted price” $P_i/cf_i$. Around a given yield level,

$$\frac{d(P_i/cf_i)}{dy} \approx -\frac{P_i}{cf_i} \times D_i$$

so higher-duration bonds’ adjusted prices move more for a given yield change.

### 23.5.2 Yield Level Effects

As rules of thumb (not laws):

- **Yields above the notional yield (e.g., above 6%):** long-duration / low-coupon bonds tend to become relatively cheaper to deliver.
- **Yields below the notional yield (e.g., below 6%):** short-duration / high-coupon bonds tend to become relatively cheaper to deliver.

Coupon structure, curve shape, and financing can dominate these tendencies.

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

At delivery, the futures settlement is the **minimum envelope** of adjusted prices:

$$F(T)=\min_i \frac{P_i(T)}{cf_i}$$

As yields move, the identity of the minimizing bond can switch. That makes the futures behave like it is **short an option**: in rallies it tends to “switch into” a lower-duration deliverable, and in sell-offs it tends to “switch into” a higher-duration deliverable. Equivalently, the *effective duration* of the futures tends to fall as yields fall.

This is the practical meaning of “negative convexity” here: when CTD is near switching, second-order effects and CTD-switch risk can dominate a hedge that looks DV01-neutral under a single assumed CTD.

### 23.5.5 Curve Shape Also Matters

With a non-flat curve, intermediate-maturity bonds can become CTD even when yields are not near the notional yield. CTD determination is therefore two-dimensional: overall yield level **and** curve shape (and, in practice, financing).

---

## 23.6 Gross Basis, Net Basis, and Carry

### 23.6.1 Gross Basis

The **gross basis** for bond $i$ at time $t$ is the clean cash–futures difference after conversion-factor adjustment:

$$\boxed{GB^i(t) = P^i(t) - cf^i \times F(t)}$$

This is the same object as the cost of delivery (away from the final delivery date it is *not* an arbitrage-free “profit,” because financing and option value matter).

- $GB\gt 0$: cash bond is rich vs CF-adjusted futures.
- $GB\lt 0$: cash bond is cheap vs CF-adjusted futures.

### 23.6.2 Net Basis and the Role of Carry

Gross basis ignores financing and coupon carry between $t$ and delivery $T$. The **net basis** incorporates carry by using the bond’s forward (or financed) price:

$$\boxed{NB^i(t) = P_{\text{fwd}}^i(t) - cf^i \times F(t)}$$

where $P_{\text{fwd}}^i(t)$ is the forward price of bond $i$ for delivery at $T$.

Using the repo/carry engine from Chapter 9, you can write the forward clean price schematically as:

$$P_{\text{fwd}}^i(t) \approx P^i(t) - \text{Carry}^i(t \to T)$$

Substituting into the net basis definition:

$$\boxed{NB^i(t) \approx GB^i(t) - \text{Carry}^i(t \to T)}$$

At the final delivery date, carry to $T$ is zero and $NB(T)=GB(T)=\text{CostDel}(T)$. The CTD has $NB^{CTD}(T)=0$.

In many practical setups, **net basis is the carry-adjusted measure used for cash–futures relative value**, and its level/dispersion across deliverables is a proxy for CTD uncertainty and delivery-option value.

### 23.6.3 Net Basis as Straddle-Like Exposure

For bonds that are close competitors for CTD, net basis behaves like an option-value object: as yields move away from the point where two deliverables are equally attractive, one bond becomes clearly non-CTD and its net basis widens. That is why CTD uncertainty shows up as basis volatility.

> **Desk Reality:** Net basis (not gross basis) is the number many cash–futures relative-value desks monitor, because it embeds funding (GC vs special) and carry.  
> **Common break:** A bond can look “cheap” on gross basis but be unattractive once you use *your actual* funding assumption (haircuts, specials, fails).  
> **What to check:** compute $GB$, carry, and $NB$ under both GC and plausible special financing, and compare implied repo to your marginal funding rate.

### 23.6.4 Basis Trade P&L

At a high level, a long-basis trade is: buy the cash bond, finance it, and short futures with the correct “tailing” so the hedge ratio is consistent with the conversion factor. Under that construction, P&L is largely driven by changes in net basis:

$$\text{PnL} \approx \frac{N_{\text{bond}}}{100}\\,[NB^i(t')-NB^i(t)]$$

where $N_{\text{bond}}$ is the bond face amount (in dollars) and $NB$ is quoted per USD100 face.

**Check (scale):** A USD100mm face position corresponds to $N_{\text{bond}}/100 = 1{,}000{,}000$ “per-100” units. So a 0.10 point move in net basis (from $NB$ changing by $-0.10$ per USD100) is about $-0.10\times 1{,}000{,}000 = -USD100{,}000$ of P&L. This is a quick sanity check on basis P&L reports.

---

## 23.7 Implied Repo Rate

### 23.7.1 Definition

The **implied repo rate** is the financing rate at which a cash-and-carry trade (buy bond, finance in repo, deliver into futures) breaks even.

Setting the delivery profit to zero:

$$(P_i(0) + AI_i(0))(1 + r_{\text{imp}} \times d/360) = cf_i \times F + AI_i(T)$$

Solving:

$$\boxed{r_{\text{imp},i} = \left(\frac{cf_i \times F + AI_i(T)}{P_i(0) + AI_i(0)} - 1\right) \frac{360}{d}}$$

**Checks (limits + scaling):** The annualization factor $360/d$ is why implied repo can look “large” over short horizons: a small delivery profit over 30–90 days annualizes into a non-trivial rate. For small profits, $r_{\text{imp}}\approx (\text{profit}/\text{dirty today})\times 360/d$. Toy: if $d=90$ days and the invoice is 0.10 per USD100 higher than the financed cost on a dirty price near 102, then $r_{\text{imp}}\approx (0.10/102)\times 360/90\approx 0.39\\%$ (about 39 bp).

### 23.7.2 Interpretation

- **If $r_{\text{imp}} \gt r_{\text{repo}}$:** The trade earns more than the cost of financing. Cash-and-carry (long basis) is attractive.
- **If $r_{\text{imp}} \lt r_{\text{repo}}$:** The trade costs more to finance than it earns. Cash-and-carry is unattractive.

Across deliverables, the bond with the highest implied repo (under your chosen delivery date and funding assumption) is often the most attractive for cash-and-carry and delivery economics. Comparing implied repo to your actual marginal funding rate is a quick sanity check for “cheap vs rich” in cash–futures relative value.

---

## 23.8 The Link to Repo and Specialness

### 23.8.1 Financing Cost Enters Through Carry

The connection between repo and futures pricing runs through carry. From Chapter 9, the forward price under repo financing is:

$$\boxed{P_{\text{fwd}} + AI(T) = (P(0) + AI(0))(1 + rd/360)}$$

Lower repo rates mean lower financing costs, which increases carry and reduces the forward price. This affects net basis and therefore CTD economics.

### 23.8.2 Specialness Can Shift CTD

Some bonds finance **special** (at repo rates below general collateral) due to scarcity and high demand to borrow that specific issue.

**Special spread** = GC rate - Special rate

If a bond in the delivery basket can be financed special, its carry improves. This can make it CTD even if its cash price is slightly higher than alternatives.

**Example logic:** Bond A costs USD101.20 to buy, but finances at 3.5% (special). Bond B costs USD100.90 but finances at 5% (GC). The lower financing cost for A may more than offset its higher price, making A the CTD.

> **Cross-reference:** Chapter 9 develops the mechanics of special repo. Here we focus on its implications for futures pricing.

---

## 23.9 Delivery Options and Their Value

### 23.9.1 Why Delivery Options Depress the Futures Price

Exact theoretical pricing is hard because the payoff depends on the short’s optimal delivery choices (which bond, which day, and sometimes when to give notice). Those choices are option-like and state-dependent.

All else equal, these options are valuable to the short. In competitive markets, the futures price reflects that value: it can trade below a naive “single-bond cost-of-carry” forward built on one assumed deliverable.

### 23.9.2 Quality Option Value

A convenient way to think about the quality option is through net basis. For each bond $i$,

$$NB^i(t)=P_{\text{fwd}}^i(t)-cf^iF(t)$$

is the carry-adjusted cost of delivering that bond (expressed as a price difference per USD100 face).

- The short will prefer the deliverable with the **lowest** net basis (equivalently, the highest implied repo).
- The dispersion of net bases across deliverables is an indicator of option value: if several bonds have similar low net basis, CTD is unstable; if one bond is clearly lowest, CTD is stable.

For near-CTD bonds, net basis can widen in either direction as rates move away from the “indifference” point where two deliverables tie—an option-like behavior that shows up as CTD-switch risk.

### 23.9.3 Timing Option: Early vs. Late Delivery

Delivery timing is a trade-off between carry and optionality:

- **Deliver early:** stop paying negative carry sooner, but give up remaining flexibility to switch deliverables and exploit timing/notice options.
- **Deliver late:** preserve the ability to switch deliverables; if carry is positive you may also benefit from holding longer, but if carry is negative you keep paying it.

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

In many contracts, the last trading day comes before the last delivery day. After trading stops, the final settlement price $\bar F$ is fixed while delivery choices remain. The short effectively holds an option to choose the minimum cost of delivery given $\bar F$:

$$CostDel_i(T)=P_i(T)-cf_i\\,\bar F.$$

If you were planning to deliver bond $i$ but bond $j$ later becomes cheaper to deliver, switching improves P&L by $CostDel_i-CostDel_j$. The window is short, so the value is usually smaller than the main quality option, but it can matter if relative values in the basket move sharply after the last trading day.

### 23.9.5 Wild Card Play

On some days there is a gap between the time the futures daily settlement is fixed and the deadline for submitting delivery notice, while the cash Treasury market continues trading. That gap creates a timing option for the short: after seeing post-settlement moves in cash prices, the short can decide whether issuing a notice is advantageous.

> **Deep Dive: The Wild Card Arbitrage**
>
> - **Settlement time:** futures daily settlement $F$ is fixed.
> - **Later:** the cash bond market still trades, and delivery notice can still be submitted.
> - **Scenario:** cash bond prices fall after futures settlement.
> - **Mechanics:** delivering after the fall lets the short buy the bond cheaper in cash while still receiving an invoice based on the earlier futures settlement.
> - **Takeaway:** this is not “free”; its expected value is embedded in the futures price.

**Check (toy scale):** If the CTD cash price drops by 1/32 (0.03125 points per USD100) after futures settlement but before the notice deadline, and the invoice remains pinned to the earlier settlement price, delivering after the drop improves economics by about 0.03125 per USD100. That is about $USD31.25$ per contract (since $N/100=1000$) — small per event, but meaningful when it happens repeatedly or on large short positions.

---

## 23.10 Futures DV01 and Hedging

### 23.10.1 DV01 Definition (Bump Object, Size, Units, Sign)

Throughout this book, we use the convention:

$$\boxed{DV01 := PV(\text{rates down }1\text{bp}) - PV(\text{base})}$$

for the **stated bump object**. Here $1\text{bp}=10^{-4}$ in rate units, and DV01 units are **currency per 1bp**. With this convention, DV01 is positive for a long position in rates (rates down → price up → PV up).

For a Treasury futures contract you must be explicit about *what is being bumped*. In this chapter, we use a CTD-based mapping:

1. **Assume a CTD bond** at time $t$ (hold CTD identity fixed for the bump).
2. **Map futures to CTD near delivery:** $F \approx P^{CTD}/cf^{CTD}$ (per USD100).
3. **Bump object:** bump the CTD yield/curve down by 1bp, reprice $P^{CTD}$, and translate that price change to $F$ using the mapping above.

This is a practical desk approximation. A more complete methodology would allow CTD to re-optimize under the bump (which can matter when net bases are tight).

### 23.10.2 Futures DV01 Approximation (CTD Mapping)

Near delivery, the futures price approximates:

$$F \approx \frac{P^{\text{CTD}}}{cf^{\text{CTD}}}$$

Holding CTD fixed and differentiating with respect to yield:

$$\frac{dF}{dy} \approx \frac{1}{cf^{\text{CTD}}} \frac{dP^{\text{CTD}}}{dy}$$

So, under the bump object above (CTD held fixed), a 1bp yield-down bump gives:

$$DV01_{\text{fut, per USD100}} \approx \frac{P^{\text{CTD}}(y-1\text{bp})-P^{\text{CTD}}(y)}{cf^{\text{CTD}}}.$$

If you denote $DV01_{\text{CTD, per USD100}} := P^{\text{CTD}}(y-1\text{bp})-P^{\text{CTD}}(y)$, then:

$$\boxed{DV01_{\text{fut, per USD100}} \approx \frac{DV01_{\text{CTD, per USD100}}}{cf^{\text{CTD}}}}$$

For a contract with face $N$ (commonly $N=USD100{,}000\Rightarrow N/100=1000$):

$$\boxed{DV01_{\text{fut, per contract}} \approx \frac{N}{100} \times \frac{DV01_{\text{CTD, per USD100}}}{cf^{\text{CTD}}}}$$

**Check (points $\leftrightarrow$ dollars):** Futures and bond prices here are “per USD100 face.” With $N=USD100{,}000$, one price point (1.00 per USD100) is $USD1000$ per contract. So a futures DV01 of 0.094 points per bp is $USD94$ per bp per contract — the same scaling as the formula above.

### 23.10.3 Hedge Ratios (Duration or DV01)

You can size a futures hedge using either duration or DV01:

- **Duration form (conceptual):** number of contracts $\propto \frac{(\text{PV of exposure})\times(\text{duration of exposure})}{(\text{PV per futures contract})\times(\text{duration of assumed CTD})}$.
- **DV01 form (most practical):**

$$\boxed{n = \frac{DV01_{\text{exposure}}}{DV01_{\text{fut}}}}$$

The key point is that $DV01_{\text{fut}}$ depends on the assumed CTD (via DV01 and conversion factor), so you must monitor it over time.

### 23.10.4 CTD Switching Risk (Why “DV01-Neutral” Hedges Break)

If CTD switches, the futures DV01 changes abruptly (because both $DV01_{CTD}$ and $cf_{CTD}$ change). A hedge that was DV01-neutral under yesterday’s CTD can become materially over- or under-hedged overnight.

This is not a subtle effect: if CTD switches between bonds with meaningfully different duration or conversion factor, the hedge ratio can jump by double-digit percentages. Monitoring near-CTDs and recalculating hedge ratios under alternative CTDs is part of operating Treasury futures hedges on a desk.

### 23.10.5 Beyond One Factor: Spot vs Repo vs Curve Shape

Treasury futures are sensitive to more than a single “parallel shift”:

- **Spot yields:** level and curve-shape moves can change relative value across the basket (and trigger CTD switches).
- **Repo/funding:** specials vs GC changes carry and can change which bond is cheapest after financing.

So there is a model-selection choice:

| Approach | Advantages | Disadvantages | When to Use |
|----------|------------|---------------|-------------|
| **One-factor (parallel shift)** | Simple, closed-form | Misses curve + repo effects | Quick hedge ratios, stable CTD |
| **Two-factor (level + slope)** | Captures curve risk | More complex | CTD near switching, curve trades |
| **Basket-level model** | Full richness of each bond | Computationally intensive | Detailed basis trading / option value |
| **One-factor + repo sensitivity** | Separates spot/repo exposures | Needs extra hedge instrument | Financed positions, stressed funding |

> **Desk Reality:** One-factor DV01 is often “good enough” when CTD is stable and funding is calm.  
> **Common break:** When CTD is near switching or repo/specials move independently, DV01-only hedges show unexplained P&L.  
> **What to check:** compute DV01 under multiple CTD assumptions, stress repo assumptions, and track net basis dispersion across near-CTDs.

---

## 23.11 Futures Rolls and Calendar Spreads

### 23.11.1 What Is a Roll?

A **calendar spread** (or **roll**) is the price difference between two futures contracts on the same underlying with different expiration months—typically the front month and the next (deferred) month.

$$\text{Roll} = F_{\text{front}} - F_{\text{deferred}}$$

The roll is also called the **calendar spread** because it represents the cost of extending a futures position from one delivery month to the next.

Most hedgers do not intend to take delivery. Instead, they roll hedges forward as contracts approach the delivery window, so the roll spread becomes a first-class object to monitor and trade.

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
- Cost per contract: USD0.50 × 1000 = USD500

> **Desk Reality:** Roll liquidity often concentrates as the front contract approaches delivery, and many traders quote the roll directly rather than legging two outrights.  
> **Common break:** Roll slippage can dominate “hedged” P&L during roll periods (it is real P&L, not noise).  
> **What to check:** compare expected roll from carry to the traded roll, and confirm CTD assumptions for *each* contract month.

### 23.11.4 When Rolls "Blow Out"

Rolls can deviate significantly from theoretical levels when:

1. **CTD differs between contracts:** If the front contract has a different CTD than the back (due to basket composition or yield changes), the roll includes both carry differential and relative value between the two CTD bonds.

2. **Squeeze in front month:** If the front-month CTD becomes extremely special or scarce (see Section 23.12), the front contract may trade rich, widening the roll.

3. **Funding/ balance-sheet effects:** shifts in financing conditions can distort repo rates and roll pricing, especially around reporting dates.

4. **Delivery uncertainty:** Near delivery, if there's uncertainty about CTD, the front contract may trade with additional premium or discount.

### 23.11.5 Roll P&L Attribution

For a hedged position that rolls, total P&L decomposes into:

$$\text{Total PnL} = \text{Cash PnL} + \text{Futures MTM} + \text{Roll Slippage}$$

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

One reason delivery baskets exist is to reduce squeeze risk. With a single deliverable, a trader could try to corner the issue and force shorts to pay distorted prices to source the bond for delivery.

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

> **Desk Reality:** In squeeze-like conditions, the CTD can finance extremely special (repo far below GC), so “normal” carry assumptions fail.  
> **Common break:** Basis-trade P&L can be dominated by funding and locate dynamics rather than yield moves.  
> **What to check:** re-run net basis / implied repo under plausible specials (or fails) assumptions and monitor deliverable supply vs open interest.

### 23.12.4 Operational Responses to Squeeze Risk

1. **Monitor deliverable supply** relative to open interest
2. **Consider alternative deliverables** if near-CTD bonds are available
3. **Adjust hedge ratios** if CTD switch becomes likely
4. **Use fails market** as last resort (delivering fails may be cheaper than paying up)
5. **Consider rolling early** to avoid squeeze premium in delivery month

---

## 23.13 Trading Case Study: The TYM0 Basis Trade

This section is an illustrative case study (Feb–May 2000) of a 10-year note futures basis trade. The purpose is to connect the chapter’s objects—net basis, CTD switching, and option protection—to a realistic P&L path. The specific numbers are dated; the mechanics are reusable.

### 23.13.1 Trade Setup (February 28, 2000)

On February 28, 2000:

- Contract: June 2000 10-year note futures (TYM0).
- A particular deliverable (the Nov 2008 4.75% bond) showed net basis around 7.5 ticks (where a “tick” is the contract’s minimum price increment).
- Trade: buy the cash bond, finance it in repo, and short futures with proper tailing.

At the time, the deliverable basket contained several bonds spanning roughly 2007–2010, and the Nov 2008 bond was CTD at initiation.

### 23.13.2 The Trade Rationale

If the “option value” embedded in the futures cheapness converges toward zero as delivery approaches, the trade earns carry-adjusted net basis. But delivery optionality makes the P&L asymmetric: rallies can push CTD toward shorter deliverables and widen net basis, producing losses.

A simple scenario analysis illustrates the shape:

- Sell-off scenario (yields +40 to +80bp): net basis compresses toward ~1 tick → gains on the basis position.
- Rally scenario (yields −60 to −80bp): net basis widens into the high single digits → losses, especially if CTD shifts away.

### 23.13.3 Option Protection

To limit rally losses, traders sometimes buy call options on the futures. A small option position can reduce drawdowns during rallies, at the cost of option premium.

### 23.13.4 What Actually Happened

Over the next few months:

- Yields rallied during March/early April and CTD shifted toward shorter deliverables.
- The Nov 2008 bond’s net basis widened; the basis position lost money.
- Call options gained and partially offset the loss.
- As yields later backed up, net basis compressed again and the basis trade recovered.

### 23.13.5 Key Lessons from the Case

1. **Basis trades can have large interim MTM** even when the “delivery convergence” thesis is ultimately right.
2. **CTD switching is a first-order risk driver:** it changes DV01 and changes which bond you are economically long.
3. **Option protection is an explicit trade-off:** pay premium to reduce convexity/CTD-switch drawdowns.
4. **Funding assumptions matter:** GC vs special can move net basis independently of yields.
5. **Tailing and unit discipline matter:** basis is per USD100; hedge is in contracts; a small scaling error becomes a big P&L error.

> **Desk Reality:** P&L on basis trades is rarely “rates only”; it is rates + funding + delivery optionality interacting.  
> **Common break:** Risk systems that hold CTD fixed can misattribute P&L when CTD is switching or repo is moving.  
> **What to check:** monitor net basis dispersion, implied repo vs actual funding, and hedge ratio under multiple CTDs.

---

## 23.14 Worked Examples

### Worked Example 1: Quote → Invoice → Implied Repo → Futures DV01

**Example Title**: Cash-and-carry snapshot for a Treasury futures contract

**Context**
- You are deciding whether a particular deliverable bond is attractive to buy-and-deliver versus its futures.
- You also want the futures DV01 to size a hedge.

**Timeline (Make Dates Concrete)**
- Trade date: 2026-02-17
- Delivery date (assumed): 2026-05-17 ($d = 90$ days)
- Bond coupon accrual period: 2026-02-15 to 2026-08-15 (semiannual)
- Next coupon payment date: 2026-08-15 (after delivery; we work via accrued interest)

**Inputs**
- Futures settlement price: $F = 112.50$ (per USD100)
- Bond clean price today: $P(0) = 101.20$ (per USD100)
- Conversion factor: $cf = 0.9012$
- Accrued interest: $AI(0) = 0.45$, $AI(T) = 1.35$ (per USD100)
- Repo rate: $r = 5.00\\%$, simple interest, ACT/360
- Contract face amount: $N = USD100{,}000$

**Outputs (What You Produce)**
- Invoice at delivery: 102.735 per USD100 (=USD102,735 per contract)
- Delivery profit (cash-and-carry snapshot, before haircuts/fees): $-0.186$ per USD100 (=-USD186 per contract)
- Implied repo rate: 4.27% (annualized ACT/360)
- Futures DV01 (CTD mapping): USD94.32 per 1bp per contract  
  (bump object: CTD yield down 1bp, CTD held fixed; bump size: 1bp $=10^{-4}$)

**Step-by-step**
1. Translate futures quote to invoice:
   - Clean delivery price: $cfF = 0.9012 \times 112.50 = 101.385$.
   - Invoice: $cfF + AI(T) = 101.385 + 1.35 = 102.735$.
2. Compute financed purchase cost to delivery:
   - Dirty today: $P(0) + AI(0) = 101.20 + 0.45 = 101.65$.
   - Repo interest: $101.65 \times 0.05 \times 90/360 = 1.271$.
   - Repayment at $T$: $101.65 + 1.271 = 102.921$.
3. Compute delivery profit:
   - $\Pi = \text{Invoice}(T) - \text{Repayment}(T) = 102.735 - 102.921 = -0.186$ per USD100.
4. Compute implied repo:
   - $r_{\text{imp}} = \left(\frac{\text{Invoice}(T)}{P(0) + AI(0)} - 1\right)\frac{360}{90} = 4.27\\%$.
5. Compute futures DV01 and hedge ratio (illustrative):
   - Assume CTD DV01 per USD100 is 0.085.
   - Futures DV01 per contract $\approx (N/100)\times 0.085/cf = 1000\times0.085/0.9012 = USD94.32$ per bp.
   - If a cash exposure has DV01 USD42,500/bp, hedge needs $42{,}500/94.32 \approx 451$ contracts.

**Cashflows (table)**
| Date | Cashflow (per USD100) | Explanation |
|---|---:|---|
| 2026-02-17 | $-101.65$ | Buy bond dirty (simplified; financed in repo) |
| 2026-05-17 | $+102.735$ | Invoice received on delivery |
| 2026-05-17 | $-102.921$ | Repo repayment (principal + interest) |
| 2026-05-17 | $-0.186$ | Net profit/loss from cash-and-carry snapshot |

**P&L / Risk Interpretation**
- Implied repo (4.27%) below your funding rate (5.00%) → the cash-and-carry loses money; the bond is not “cheap enough” versus the futures under GC funding.
- If the bond can be financed special, carry improves and the economics can flip.
- DV01 mapping assumes CTD stays the same; if CTD is near switching, hedge ratios can jump.

**Sanity Checks**
- Units: all prices/basis are per USD100; per-contract scaling is $N/100=1000$.
- Sign: rates down 1bp should increase $P^{CTD}$ and $F$, so DV01 is positive for long futures.
- Limit: if you plug $r=r_{imp}$ into the financing step, $\Pi$ should be ~0.

### Conventions for Examples

Unless otherwise stated:
- Prices are per USD100 face
- Repo uses simple interest with ACT/360
- Futures contract size $N = USD100{,}000$ (so $N/100 = 1000$)
- Invoice price formula: $\text{Invoice} = cf \times F + AI$

### Example A: Invoice Price Calculation

**Inputs:**
- Futures settlement: $F = 112.50$
- Conversion factor: $cf = 0.9012$
- Accrued at delivery: $AI(T) = 1.35$

**Calculation:**

Delivery price: $cf \times F = 0.9012 \times 112.50 = 101.385$

Invoice price: $101.385 + 1.35 = 102.735$ USD per USD100 face

Per contract: $102.735 \times 1000 = 102{,}735$ USD

### Example B: Delivery Economics

Buy bond today, finance in repo, deliver at $T$.

**Inputs:**
- Bond clean price: $P(0) = 101.20$
- Accrued today: $AI(0) = 0.45$
- Repo rate: $r = 5.00\\%$
- Days to delivery: $d = 90$
- Invoice at delivery: $\text{Invoice}(T) = 102.735$ (from Example A)

**Financing repayment:**

$$101.65 \times \left(1 + \frac{0.05 \times 90}{360}\right) = 101.65 \times 1.0125 = 102.921$$

**Delivery profit:**

$$\Pi = 102.735 - 102.921 = -0.186 \text{ per USD100}$$

Per contract: $-USD186$

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

Financing cost: $101.65 \times 0.05 \times 90/360 = 1.271$ USD

$$\text{Carry} = 0.90 - 1.271 = -0.371$$

**Net basis:**

$$NB = GB - \text{Carry} = -0.185 - (-0.371) = +0.186$$

Positive net basis means cash-and-carry loses money at this financing rate (consistent with Example B).

### Example F: Implied Repo Rate

**Inputs:**
- Dirty price today: 101.65 USD
- Invoice at delivery: 102.735 USD
- Days: $d = 90$

**Break-even condition:**

$$101.65(1 + r_{\text{imp}} \times 90/360) = 102.735$$

$$r_{\text{imp}} = \left(\frac{102.735}{101.65} - 1\right) \times \frac{360}{90} = 0.01067 \times 4 = 4.27\\%$$

If you can finance below 4.27%, cash-and-carry is profitable. At 5% GC, it loses money.

### Example G: Futures DV01 and Hedge Ratio

**Inputs:**
- Cash position: long USD50 million face, DV01 = 0.085 per USD100
- CTD: Bond A with DV01 = 0.085, cf = 0.9012

**Cash DV01 in dollars:**

$$0.085 \times \frac{50{,}000{,}000}{100} = USD42{,}500 \text{ per bp}$$

**Futures DV01 per contract:**

$$1000 \times \frac{0.085}{0.9012} = 1000 \times 0.0943 = USD94.32 \text{ per bp}$$

**Hedge ratio:**

$$n = \frac{42{,}500}{94.32} \approx 451 \text{ contracts}$$

**If CTD switches to C** (DV01 = 0.095, cf = 0.8420):

$$\text{Futures DV01} = 1000 \times \frac{0.095}{0.8420} = USD112.83 \text{ per bp}$$

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
- Net cost per contract: $0.35 \times 1000 = 350$ USD
- Total roll cost: $100 \times 350 = 35{,}000$ USD

**Sanity check:** The roll should approximate carry from Mar to Jun delivery. If carry is approximately $USD0.35$ per USD100 over 3 months, this is consistent.

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

1. Treasury futures are deliverable-basket contracts; the short holds delivery options (bond choice, timing, and sometimes notice timing).
2. Conversion factors map a single futures quote into bond-specific delivery economics; invoice at delivery is $cf\cdot F + AI$.
3. Cost of delivery is $P - cf\cdot F$ (clean); CTD minimizes it. At final delivery, $F(T)=\min_i P_i(T)/cf_i$ and the CTD cost of delivery is 0.
4. CTD can switch with yield level, curve shape, and financing; this makes futures risk state-dependent and creates “negative convexity” intuition.
5. Gross basis $GB=P-cfF$ is the clean cash–futures spread; net basis $NB=P_{fwd}-cfF\approx GB-\text{carry}$ is carry-adjusted and desk-relevant.
6. Implied repo is the break-even financing rate for buy-and-deliver; compare it to your marginal funding (GC vs special).
7. Delivery options transfer value from long to short and explain why exact theoretical pricing is harder than a single-bond cost-of-carry forward.
8. Futures DV01 depends on the bump object and the assumed CTD mapping; under a CTD-held-fixed mapping, $DV01_{fut}\approx DV01_{CTD}/cf$.
9. Hedges break when CTD switches; re-run DV01 and hedge ratios under alternative CTDs.
10. Rolls and squeezes are funding-and-delivery phenomena; carry and specials can dominate yield moves near delivery.

---

## Key Concepts

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| Conversion factor | Bond price at 6% yield per USD1 face; scales invoice price | Links futures price to individual bond delivery |
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

## Notation

| Symbol | Meaning | Units / Convention |
|---|---|---|
| $t,T$ | valuation time; delivery time | date (or year-fraction if stated) |
| $F(t)$ | futures settlement price | quoted per USD100 notional; clean-like |
| $cf_i$ | conversion factor for bond $i$ | unitless; exchange-published |
| $P_i(t)$ | clean cash price of bond $i$ | dollars per USD100 face |
| $AI_i(t)$ | accrued interest of bond $i$ | dollars per USD100 face; $P_{dirty}=P_{clean}+AI$ |
| $Invoice_i(t)$ | delivery cash amount per USD100 | $cf_iF(t)+AI_i(t)$ |
| $CostDel_i(t)$ | delivery economics per USD100 | $P_i(t)-cf_iF(t)$ (clean) |
| $GB^i(t)$ | gross basis per USD100 | $P_i(t)-cf_iF(t)$ |
| $NB^i(t)$ | net basis per USD100 | $P_{\text{fwd}}^i(t)-cf_iF(t)\approx GB-\text{carry}$ |
| $r_{\text{repo}}$ | repo rate used for financing | annualized; simple; ACT/360 in examples |
| $r_{\text{imp}}$ | implied repo rate | annualized; ACT/360 in examples |
| $d$ | days to delivery | days (calendar) |
| $N$ | futures contract face | dollars; examples use $N=USD100{,}000$ |
| $DV01$ | PV sensitivity scalar | dollars per 1bp; convention: $PV(\text{rates down }1\text{bp})-PV(\text{base})$ for the stated bump object |
| $DV01_{fut}$ | futures DV01 | dollars per 1bp per contract; bump object in this chapter: CTD yield down 1bp, CTD held fixed |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is a deliverable-basket futures contract? | A futures where the short can choose which eligible bond to deliver (and often when) within a delivery window |
| 2 | Who owns the delivery options? | The short position holder |
| 3 | What is a conversion factor $cf_i$? | A fixed scale that maps the futures quote into a bond-specific delivery price for bond $i$ (often defined off a notional 6% yield, semiannual) |
| 4 | What is the invoice price formula? | $Invoice_i(t)=cf_iF(t)+AI_i(t)$ |
| 5 | Why is accrued interest added at delivery? | Cash bonds settle dirty; accrued interest is part of the cash amount |
| 6 | Write the cost-of-delivery formula | $CostDel_i(t)=P_i(t)-cf_iF(t)$ (clean) |
| 7 | Why does accrued interest cancel in cost of delivery? | It is added to both the dirty purchase cost and the invoice received |
| 8 | Define CTD | The bond that minimizes cost of delivery (equivalently, minimizes net basis under a funding assumption) |
| 9 | What is the futures price at final delivery in a no-arbitrage setting? | $F(T)=\min_i P_i(T)/cf_i$ |
| 10 | What does gross basis measure? | Clean cash price minus CF-adjusted futures price: $GB=P-cfF$ |
| 11 | What does net basis measure? | Carry-adjusted cash–futures difference: $NB=P_{fwd}-cfF\approx GB-\text{carry}$ |
| 12 | What does implied repo measure? | The break-even financing rate for buy-and-deliver given $F$, $cf$, and accrued interest |
| 13 | When is cash-and-carry attractive? | When implied repo exceeds your marginal funding rate (under the same assumptions) |
| 14 | Name the main short-owned delivery options | Quality (bond choice), timing (delivery date), end-of-month switching, notice-timing (“wild card” style) |
| 15 | Why can delivery options depress the futures price? | The long effectively pays for options owned by the short, so the futures can trade below a naive single-bond forward |
| 16 | What is the DV01 convention in this book? | $DV01=PV(\text{rates down }1\text{bp})-PV(\text{base})$, positive for long rates risk |
| 17 | What is being bumped for futures DV01 in this chapter? | CTD yield (or CTD pricing curve) down 1bp, holding CTD identity fixed |
| 18 | Give the CTD-mapped futures DV01 approximation | $DV01_{fut,USD100}\approx DV01_{CTD,USD100}/cf_{CTD}$ |
| 19 | Give the DV01 hedge ratio | $n_{\text{contracts}}=DV01_{exposure}/DV01_{fut}$ |
| 20 | What is CTD switching risk? | Risk that the CTD changes, causing DV01 and hedge ratios to jump |
| 21 | What does “special” repo mean? | A bond finances below GC because it is scarce/in demand to borrow |
| 22 | How can specialness change CTD? | Lower financing cost improves carry, lowers net basis, and can make a bond cheapest after financing |
| 23 | What is a roll (calendar spread)? | $F_{front}-F_{deferred}$: the cost of extending a futures position to a later delivery month |
| 24 | What can move the roll away from “carry”? | Different CTDs across months, funding shifts, squeeze risk, and delivery uncertainty |
| 25 | What is a squeeze (in this context)? | CTD scarcity → extreme specialness → distorted basis/roll near delivery |

---

## Mini Problem Set
1. (Compute) Compute invoice price given $F = 108.50$, $cf = 0.8750$, $AI = 2.10$.
2. (Compute) Given three deliverables with $(P,cf)=[(98.50,0.92),(105.00,0.98),(91.20,0.85)]$ and $F = 106.00$, identify CTD using $P-cfF$.
3. (Compute) Compute gross basis for Bond A: $P = 102.50$, $cf = 0.9500$, $F = 108.00$.
4. (Compute) Compute carry for 60 days: $P + AI(0)=103.20$, $AI(T)-AI(0)=1.00$, $r=4.50\\%$, ACT/360 simple.
5. (Compute) Compute net basis from gross basis $-0.10$ and carry $0.226$.
6. (Compute) Compute implied repo: dirty today $=103.20$, invoice at delivery $=103.85$, $d=60$ days.
7. (Compute) Cash DV01 $=USD30{,}000/\text{bp}$. CTD DV01 $=0.072$ per USD100, $cf=0.8500$. Compute futures DV01 per contract and the hedge ratio.
8. (Compute) Roll: $F_{Mar}=112.50$, $F_{Jun}=112.15$. Compute roll and the cost to roll 100 short contracts.
9. (Concept) Explain why the futures price is not a pure cost-of-carry forward price on a single bond.
10. (Desk) Describe how a CTD switch can create P&L on an otherwise DV01-neutral hedge.
11. (Desk) Construct a scenario where gross basis is negative but net basis is positive. Why might a basis trade lose money?
12. (Desk) Write a daily monitoring checklist for a live Treasury futures hedge (CTD status, net basis, implied repo, roll, specials).

### Solution Sketches (Selected)
1. Invoice $=0.8750\times108.50+2.10=97.04$ per USD100. Per contract: $97.04\times1000=USD97{,}040$.
2. Compute $P-cfF$: 0.98, 1.12, 1.10 → CTD is Bond 1 (minimum).
3. $GB=102.50-0.95\times108.00=-0.10$ per USD100.
6. $r_{imp}=(103.85/103.20-1)\times 360/60\approx 3.78\\%$.
7. Futures DV01 per contract $=1000\times(0.072/0.85)=USD84.71/\text{bp}$. Hedge $\approx 30{,}000/84.71\approx 354$ contracts.

---

## References

- Tuckman & Serrat, *Fixed Income Securities: Tools for Today’s Markets*, Treasury futures chapter/sections on mechanics, CTD, gross/net basis, and delivery options.
- Hull, *Options, Futures, and Other Derivatives*, sections on Treasury bond futures (conversion factors, CTD) and hedging with bond futures (duration/DV01 logic, delivery-option intuition).
- Neftci, *Principles of Financial Engineering*, sections on futures arbitrage / implied repo and repo special vs general collateral.
- Musiela & Rutkowski, *Martingale Methods in Financial Modelling*, discussion of Treasury bond futures delivery factors and invoice-price mechanics.
- Elton, Gruber, Brown & Goetzmann, *Modern Portfolio Theory and Investment Analysis*, section on Treasury bond futures and the role of conversion factors.
