# Chapter 29: FX Spot and Forwards — Pricing via Interest Differentials

---

## Introduction

EUR/USD spot is 1.10. The 1-year forward is 1.12. Is the market predicting that the euro will appreciate against the dollar?

The intuitive answer—"yes, the forward is higher, so the market expects EUR to rise"—is wrong. The forward rate has almost nothing to do with market expectations of future spot rates. Instead, the forward is pinned down by a fundamental no-arbitrage relationship: **covered interest parity (CIP)**. The forward differs from spot because interest rates differ across currencies. If you could earn more by investing in dollars than in euros, that advantage must be exactly offset by the forward FX rate—otherwise, you could lock in a risk-free profit.

This insight has profound implications for anyone managing cross-currency exposures. When a U.S. investor hedges a European bond with FX forwards, they are not betting on currency direction—they are locking in the interest differential. When a corporation hedges a future foreign-currency payable, the "cost" of hedging is not some penalty; it *is* the interest differential, which reflects the opportunity cost of holding one currency versus another. Understanding FX forwards means understanding that currency markets and interest rate markets are inextricably linked.

Hull presents a vivid example of corporate hedging: "ImportCo, a company based in the United States, knows that it will have to pay £10 million on August 21, 2020, for goods it has purchased from a British supplier... ImportCo could hedge its foreign exchange risk by buying pounds from the financial institution in the 3-month forward market at 1.2225. This would have the effect of fixing the price to be paid to the British exporter at $12,225,000." The forward eliminates uncertainty—but at what "cost"? The answer lies in understanding CIP.

This chapter covers:

1. **FX spot and quote conventions** (Section 29.1) — what $S^{D/F}$ means and why quote direction matters
2. **Settlement mechanics and value dates** (Section 29.2) — when cash actually moves, what "spot" means operationally
3. **The FX forward contract** (Section 29.3) — structure and payoff at maturity
4. **Covered interest parity** (Section 29.4) — deriving $F = S \cdot P_F / P_D$ from no-arbitrage
5. **Forward points and quoting conventions** (Section 29.5) — the mechanics of $F - S$ and how traders quote
6. **Why forward ≠ expected future spot** (Section 29.6) — the critical distinction
7. **Valuing an FX forward** (Section 29.7) — PV in domestic currency terms
8. **Hedging with FX forwards** (Section 29.8) — locking in domestic value of foreign cashflows
9. **Risk sensitivities** (Section 29.9) — FX delta and rates exposure
10. **Non-Deliverable Forwards (NDFs)** (Section 29.10) — restricted currencies and fixing risk
11. **P&L attribution** (Section 29.11) — decomposing FX forward returns
12. **Rolling positions: Tom/Next swaps** (Section 29.12) — brief introduction; full treatment in Chapter 30

We connect forward pricing to Chapter 21 (cross-currency curves), which extends these ideas to longer maturities where cross-currency basis becomes important, and preview Chapter 30 (FX swaps and cross-currency swaps), which builds on these foundations for multi-period structures.

---

## 29.1 Spot FX and Quote Conventions

### 29.1.1 What the Spot Rate Represents

A spot FX rate $S_t^{D/F}$ is the price of one unit of foreign currency in units of domestic currency. If $D =$ USD and $F =$ EUR, then $S^{\text{USD/EUR}} = 1.10$ means one euro costs 1.10 dollars.

Hull notes that exchange rates are "normally quoted as the number of units of the currency that are equivalent to one U.S. dollar," with some exceptions (notably GBP and EUR, which are quoted as USD per unit). In *Options, Futures, and Other Derivatives*, he adopts the convention of defining $S_0$ as "the current spot price in U.S. dollars of one unit of the foreign currency." This chapter follows the same "domestic per foreign" convention consistently: $S_t^{D/F}$ means domestic currency units per one foreign unit.

> **Why this matters:** Risk systems must store both the currency pair and the quote direction. Mis-storing the inverse is a common source of sign errors in P&L, hedge ratios, and position reports. A system that treats EUR/USD as "EUR per USD" when the market quotes it as "USD per EUR" will produce inverted hedge ratios.

### 29.1.2 Inverting the Quote

The inverse quote simply flips the numerator and denominator:

$$\boxed{S_t^{F/D} = \frac{1}{S_t^{D/F}}}$$

If EUR/USD (USD per EUR) is 1.20, then USD/EUR (EUR per USD) is $1/1.20 = 0.8333$.

This seems trivial, but the inversion becomes critical when applying CIP formulas. As we will see, inverting the spot quote also inverts the ratio of discount factors in the forward pricing formula. Getting this wrong is one of the most common implementation errors in cross-currency systems.

> **Desk Reality: FX Quote Conventions**
>
> The FX market has evolved specific quote conventions for major pairs:
>
> | Convention | Example Pairs | Quote Meaning |
> |------------|--------------|---------------|
> | **Base/Quote** | EUR/USD, GBP/USD | USD per 1 unit of EUR or GBP |
> | **Quote/Base** | USD/JPY, USD/CHF | JPY or CHF per 1 USD |
>
> When a trader says "EUR/USD is bid at 1.0950," they mean someone will buy euros at the rate of 1.0950 USD per EUR. When they say "Dollar-Yen is offered at 150.25," they mean someone will sell dollars at 150.25 JPY per USD.
>
> **The "Big Figure" shorthand:** Traders often drop the leading digits. "Cable is fifty-two, fifty-five" means GBP/USD is bid at 1.2752, offered at 1.2755. The "big figure" (1.27) is understood from context.

### 29.1.3 Cross Rates and Triangular Consistency

When trading crosses (non-USD pairs), the market derives rates from the USD legs:

$$S^{EUR/GBP} = \frac{S^{USD/GBP}}{S^{USD/EUR}}$$

If USD/GBP = 1.2750 and USD/EUR = 1.0900, then EUR/GBP = 1.2750/1.0900 = 1.1697.

Arbitrage keeps cross rates consistent. If the calculated cross differs from the quoted cross by more than transaction costs, traders profit by buying cheap and selling dear across the three pairs. This **triangular arbitrage** keeps the currency market internally consistent.

> **Practitioner Note:** Most retail and interbank platforms quote crosses directly, but the underlying liquidity is in the USD pairs. When you trade EUR/GBP, the dealer often executes EUR/USD and USD/GBP legs, earning the cross spread. This explains why cross pairs typically have wider bid-ask spreads than major USD pairs.

---

## 29.2 Settlement Mechanics and Value Dates

Understanding exactly *when* FX transactions settle is essential for pricing short-dated forwards and for operational accuracy. This section covers the practical mechanics that textbooks often omit.

### 29.2.1 Spot Value Date: T+2 (Mostly)

> **Practitioner Note:** The following settlement conventions are standard market practice but are not specified in Hull or Andersen. I've marked this section as Category B (Claude-extended).

The **spot value date** is when the currencies actually exchange hands for a spot transaction. For most currency pairs:

| Pair Type | Settlement | Examples |
|-----------|------------|----------|
| **Major pairs** | T+2 | EUR/USD, GBP/USD, USD/JPY, AUD/USD |
| **North American** | T+1 | USD/CAD, USD/MXN |
| **Certain emerging** | Varies | Some settle T+1 or T+3 |

**T+2 means:** If you trade on Monday, settlement occurs on Wednesday (assuming no holidays). Both currencies must settle on a business day in *both* financial centers.

### 29.2.2 How Value Dates Are Determined

The value date determination follows a specific protocol:

1. **Start from trade date**: Count forward the required number of business days
2. **Check both calendars**: The value date must be a business day in both currency centers
3. **If not valid in both**: Roll to the next day that *is* valid in both

**Example:** A EUR/USD trade done on a Friday would normally settle Tuesday (T+2). But if Tuesday is a U.S. holiday (say, Presidents' Day), settlement rolls to Wednesday. However, if Wednesday is a TARGET holiday in Europe, settlement might roll to Thursday.

> **Desk Reality: Holiday Calendar Headaches**
>
> Cross-border FX operations require maintaining accurate holiday calendars for every financial center. Common pain points:
>
> - **Year-end:** Different markets close on different days around Christmas and New Year
> - **Regional holidays:** Good Friday is a holiday in London but not New York
> - **Moving holidays:** Lunar New Year, Eid, and Golden Week dates vary annually
>
> Most FX desks use Bloomberg's calendar service (CALY function) or similar to determine valid settlement dates. Getting this wrong causes settlement fails.

### 29.2.3 Forward Value Dates: Spot + Tenor

Forward value dates are calculated as **spot date + tenor**:

- **1-week forward:** Spot + 7 calendar days (adjusted to business day)
- **1-month forward:** Spot + 1 month (same day of month, adjusted)
- **3-month forward:** Spot + 3 months (adjusted)

The adjustment follows the **Modified Following** convention: roll to the next business day unless that crosses a month boundary, in which case roll backward.

**Example:** If spot is January 30 and you trade a 1-month forward:
- Naive calculation: February 30 (doesn't exist)
- Adjusted: February 28 (or 29 in leap year)—the last business day of February

### 29.2.4 Broken Dates

A **broken date** (or odd date) is any date that doesn't fall on a standard tenor (1W, 1M, 2M, 3M, etc.). For example, a forward settling in 47 days is a broken date.

Pricing broken dates requires interpolating forward points between the surrounding standard tenors. Linear interpolation is common, though some desks use more sophisticated methods for very short or very long broken dates.

### 29.2.5 Same-Day and Tomorrow Value

Beyond spot, the FX market trades shorter tenors:

| Tenor | Value Date | Use Case |
|-------|------------|----------|
| **TOD** (Today) | Trade date | Urgent settlement, typically higher cost |
| **TOM** (Tomorrow) | T+1 | One day before spot |
| **Spot** | T+2 | Standard |

The pricing from TOD to spot follows the same interest differential logic, but applied to very short periods. The difference between TOM and spot prices (the **Tom/Next swap**) is used to roll positions—covered briefly in Section 29.12 and fully in Chapter 30.

---

## 29.3 The FX Forward Contract

### 29.3.1 Contract Structure

A forward contract is an agreement made today to exchange assets at a future time $T$. An FX forward specifically exchanges two currencies:

- **Long FX forward (buy foreign):** At maturity $T$, receive $N_F$ units of foreign currency and pay $K \cdot N_F$ units of domestic currency, where $K$ is the delivery price (strike) agreed at inception.

Hull illustrates the corporate motivation clearly: "ImportCo could hedge its foreign exchange risk by buying pounds from the financial institution in the 3-month forward market at 1.2225. This would have the effect of fixing the price to be paid to the British exporter at $12,225,000."

The key feature is that $K$ is fixed at trade inception, while the spot rate $S_T$ at maturity is unknown. The forward eliminates the uncertainty about how many domestic dollars will be needed.

> **Analogy: The Storage Locker**
>
> Why is the Forward Rate different from the Spot Rate? Is it a prediction? No.
> Think of buying a Forward as **putting money in a foreign storage locker**.
>
> 1. **Spot**: You exchange USD for EUR today.
> 2. **Storage**: You put the EUR in a bank account (a "locker") for 1 year.
> 3. **Interest**: The locker *pays* you interest (the EUR rate).
> 4. **Forward**: The Forward Rate is just the price that accounts for the fact that you own a "fuller locker" at the end of the year.
>
> Buying a Forward is mathematically identical to buying Spot and earning interest. Therefore, the price *must* reflect that interest difference.

### 29.3.2 Payoff at Maturity

If you translate the payoff into domestic currency units at maturity:

$$\boxed{\text{Domestic-equivalent payoff at } T = N_F(S_T - K)}$$

This is the standard forward payoff: you agreed to buy foreign currency at $K$, but the spot rate at maturity is $S_T$. If $S_T > K$, the forward is profitable (you bought cheap); if $S_T < K$, you overpaid relative to spot.

**Unit check:** Both $S_T$ and $K$ are in $D/F$ units, so $(S_T - K) \cdot N_F$ is in domestic currency. ✓

Hull notes the parallel for the counterparty: "Consider next another U.S. company, which we will refer to as ExportCo, that is exporting goods to the United Kingdom and, on May 21, 2020, knows that it will receive £30 million 3 months later. ExportCo can hedge its foreign exchange risk by selling £30 million in the 3-month forward market." ExportCo takes the opposite position—short the forward (sell foreign forward)—and locks in its dollar proceeds.

---

## 29.4 Covered Interest Parity: The Core Pricing Relationship

### 29.4.1 The No-Arbitrage Argument

Hull presents the fundamental pricing relationship for currency forwards in Chapter 5:

$$\boxed{F_0 = S_0 e^{(r - r_f)T}}$$

where $r$ is the domestic risk-free rate and $r_f$ is the foreign risk-free rate (both continuously compounded).

The intuition comes from treating foreign currency as an asset providing a yield. As Hull explains: "A foreign currency has the property that the holder of the currency can earn interest at the risk-free interest rate prevailing in the foreign country. For example, the holder can invest the currency in a foreign-denominated bond." Thus, "the foreign currency can be regarded as an investment asset paying a known yield. The yield is the risk-free rate of interest in the foreign currency."

This is a powerful insight: **FX forward pricing is just dividend-adjusted forward pricing**, with the foreign rate playing the role of the dividend yield.

### 29.4.2 The Replication Argument

Hull provides an explicit arbitrage argument illustrated in Figure 5.1 of his text. Consider two strategies to convert 1,000 units of foreign currency into domestic currency at time $T$:

**Strategy A (Use forward):**
1. Enter a forward contract to sell foreign currency at rate $F_0$
2. At $T$, convert the 1,000 foreign using the forward: receive $1,000 \cdot F_0$ domestic

**Strategy B (Spot + invest):**
1. Invest 1,000 foreign at the foreign rate $r_f$: grows to $1,000 \cdot e^{r_f T}$ at $T$
2. Simultaneously borrow the domestic-currency equivalent today: $1,000 \cdot S_0$
3. The domestic borrowing grows to $1,000 \cdot S_0 \cdot e^{rT}$ at $T$

For no arbitrage, both strategies must give the same terminal domestic amount:

$$1,000 \cdot e^{r_f T} \cdot F_0 = 1,000 \cdot S_0 \cdot e^{rT}$$

Solving: $F_0 = S_0 e^{(r - r_f)T}$.

This is **covered** interest parity because the FX conversion at maturity is locked in via the forward—there is no currency risk in the replication. Hull explicitly calls this "the well-known interest rate parity relationship from international finance."

### 29.4.3 Worked Arbitrage Example (from Hull)

Hull provides a concrete numerical example:

> Suppose that the 2-year interest rates in Australia and the United States are 3% and 1%, respectively, and the spot exchange rate is 0.7500 USD per AUD. From equation (5.9), the 2-year forward exchange rate should be:
>
> $$0.7500 \times e^{(0.01-0.03) \times 2} = 0.7206$$

If the market forward were instead 0.7000 (too low), Hull shows the arbitrage:

1. Borrow 1,000 AUD at 3% for 2 years, convert to 750 USD and invest at 1%
2. Enter a forward to buy 1,061.84 AUD (the amount owed) for $1,061.84 \times 0.7000 = \$743.29$
3. The 750 USD grows to $765.15. Pay $743.29 to settle the forward, repay the AUD loan
4. **Risk-free profit: $765.15 - 743.29 = \$21.87$ per 1,000 AUD**

Hull notes: "If this does not sound very exciting, consider following a similar strategy where you borrow 100 million AUD!"

### 29.4.4 The Discount Factor Form

A more robust formulation uses discount factors, avoiding ambiguity about compounding conventions. From Andersen and Piterbarg's multi-currency framework:

$$\boxed{F(t,T) = S_t \frac{P_F(t,T)}{P_D(t,T)}}$$

where:
- $P_D(t,T)$ is the domestic discount factor (price of a domestic zero-coupon bond paying 1 at $T$)
- $P_F(t,T)$ is the foreign discount factor

Andersen and Piterbarg use this structure explicitly in cross-currency curve construction, noting that "the market for foreign exchange (FX) forwards and cross-currency basis swaps imposes certain arbitrage constraints that must be considered in the curve construction exercise."

**Why prefer discount factors?** The discount factor form is convention-robust: it works regardless of whether rates are quoted as simple, semiannual, or continuous. The formula $F = S \cdot P_F / P_D$ simply says: the forward equals spot scaled by the ratio of "present value of 1 unit of foreign" to "present value of 1 unit of domestic."

**Unit check:** $P_F / P_D$ is dimensionless, so $F$ inherits the units of $S$ ($D/F$). ✓

**Reconciliation with rate form:** Under continuous compounding, $P_D = e^{-rT}$ and $P_F = e^{-r_f T}$, so:

$$\frac{P_F}{P_D} = \frac{e^{-r_f T}}{e^{-rT}} = e^{(r - r_f)T}$$

Multiplying by $S$ recovers Hull's formula.

---

## 29.5 Forward Points: What They Reveal

### 29.5.1 Definition and Mechanics

Forward points are the difference between the forward rate and the spot rate:

$$\text{Forward points} = F(t,T) - S_t$$

Using the CIP formula:

$$F(t,T) - S_t = S_t \left( \frac{P_F(t,T)}{P_D(t,T)} - 1 \right)$$

This shows that forward points are driven entirely by the ratio of discount factors—which reflects the interest rate differential between the two currencies.

### 29.5.2 Premium vs Discount

- **Forward premium (in $D/F$ quotes):** If $F > S$, the foreign currency trades at a forward premium. This occurs when $P_F / P_D > 1$, meaning domestic rates are higher than foreign rates (equivalently, $P_D < P_F$ for the same maturity).

- **Forward discount (in $D/F$ quotes):** If $F < S$, the foreign currency trades at a forward discount. This occurs when foreign rates exceed domestic rates.

Hull provides market evidence: "For all the currencies considered in the table, short-term interest rates were lower than on the U.S. dollar. This corresponds to the $r > r_f$ situation and explains why the settlement futures prices of these currencies increase with maturity."

> **Intuition:** If you can earn more interest holding dollars than euros, then locking in a future EUR purchase via a forward must cost you something—you give up the higher domestic interest. That cost shows up as a forward premium on EUR (you pay more USD per EUR in the forward than at spot).

> **Rule of Thumb: The Forward Point Check**
>
> - **High-Yielding Currency** = Trades at a **Forward Discount**
>   - Price of the high-yielder (in terms of the low-yielder) will be LOWER in the forward
>   - *Why?* You get paid interest to hold it, so the forward price is lower to offset that advantage
>
> - **Low-Yielding Currency** = Trades at a **Forward Premium**
>   - Price will be HIGHER in the forward
>   - *Why?* You pay a penalty (negative carry) to hold it vs the high-rate currency
>
> **Visual: The Conveyor Belt**
> Imagine Spot is at the start of a conveyor belt.
> - The **Motor** is the Interest Rate Differential
> - If Foreign Rate > Domestic Rate, the motor pulls the price **Down** (Discount)
> - If Foreign Rate < Domestic Rate, the motor pushes the price **Up** (Premium)

### 29.5.3 Quoting Conventions: Pips and Points

> **Practitioner Note:** Forward points are typically quoted in **pips**. For most major pairs, 1 pip = 0.0001 (the 4th decimal). For JPY pairs, 1 pip = 0.01 (the 2nd decimal). Always confirm the convention for the specific pair and venue.

Forward points are typically quoted in **pips** (percentage in point). For most currency pairs:

- 1 pip = 0.0001 (fourth decimal place)
- For JPY pairs: 1 pip = 0.01 (second decimal place)

| Pair | Spot | 3M Forward | Forward Points (pips) |
|------|------|------------|----------------------|
| EUR/USD | 1.0850 | 1.0875 | +25 |
| USD/JPY | 150.00 | 149.50 | -50 |
| GBP/USD | 1.2650 | 1.2625 | -25 |

**Reading a forward points table:**
- Positive points = Forward > Spot = Foreign at premium
- Negative points = Forward < Spot = Foreign at discount

**Bid-ask on forward points:** Dealers quote two-way prices on points, just like spot:

> EUR/USD 3M points: 23/27

This means:
- Dealer buys EUR forward (sells USD forward) at spot + 23 pips
- Dealer sells EUR forward (buys USD forward) at spot + 27 pips

The forward bid-ask incorporates both spot bid-ask and points bid-ask.

### 29.5.4 Quote Inversion and Sign Conventions

Hull cautions that "conventions can differ" in how basis and forward points are expressed. A critical point: **inverting the quote flips the sign interpretation.**

If $F^{D/F} > S^{D/F}$ (forward premium in domestic-per-foreign terms), then:

$$F^{F/D} = \frac{1}{F^{D/F}} < \frac{1}{S^{D/F}} = S^{F/D}$$

In the inverted quote, the forward is at a *discount*. Same economics, different sign—depending on which way you read the quote.

---

## 29.6 Why Forward ≠ Expected Future Spot

This distinction is fundamental and often misunderstood. The forward rate tells you nothing about where the market "expects" spot to be at maturity.

### 29.6.1 The Arbitrage Explanation

The forward rate is determined entirely by **current** spot and **current** interest rates:

$$F = S \cdot e^{(r - r_f)T}$$

No expectations about future spot rates enter this equation. The forward is the "fair" exchange rate that prevents risk-free arbitrage between borrowing, lending, and FX markets—*today*.

### 29.6.2 The Forward Rate Bias Puzzle

Hull poses an important question: "It is sometimes argued that a forward exchange rate is an unbiased predictor of future exchange rates. Under what circumstances is this so?"

Empirically, the answer is: almost never. This is the famous **forward rate bias** (or forward premium puzzle):

> **Practitioner Note:** Academic studies consistently find that high-interest-rate currencies tend to *appreciate* rather than depreciate as the forward would suggest. The "carry trade" (borrow low-rate, invest high-rate) has been profitable on average, despite the forward predicting the high-rate currency should weaken.

**Key insight:** CIP is a no-arbitrage relationship that *must* hold (within transaction costs). **Uncovered interest parity** (UIP)—the hypothesis that the forward equals expected future spot—is an economic theory that empirically often fails.

### 29.6.3 Practical Implication

When someone says "the forward predicts EUR appreciation," they are confusing arbitrage pricing with forecasting:

- **Correct:** The forward is higher than spot because USD rates exceed EUR rates
- **Incorrect:** The forward predicts EUR will appreciate

The forward embeds *financing costs*, not forecasts. A hedger using the forward is not making a directional bet—they are locking in the interest differential.

> **Desk Reality: Don't Confuse Hedging Cost with "Prediction"**
>
> A corporate treasurer hedging EUR receivables with a forward is often asked: "Why pay up for the forward when the market is 'predicting' EUR strength?"
>
> The correct response: "The forward doesn't predict anything. The premium reflects that I'm giving up higher USD interest by locking in EUR. If I wanted to speculate on EUR appreciation, I'd leave the exposure unhedged—but that's not my job."

---

## 29.7 Valuing an FX Forward: PV in Domestic Currency

### 29.7.1 The Two-Leg Decomposition

An FX forward has two cashflows at maturity:
- Receive $N_F$ units of foreign currency
- Pay $K \cdot N_F$ units of domestic currency

Each leg is valued in domestic currency, then netted.

### 29.7.2 Domestic PV Formula

Hull provides the general valuation formula for a forward on an asset with yield $q$ (equation 5.7):

$$f = S_0 e^{-qT} - K e^{-rT}$$

For FX, setting $q = r_f$ (foreign rate as the "yield" on foreign currency):

$$f = S_0 e^{-r_f T} - K e^{-rT}$$

In discount factor notation:

**PV of receiving $N_F$ foreign at $T$:**
$$PV_{\text{receive}} = N_F \cdot S_t \cdot P_F(t,T)$$

**PV of paying $K \cdot N_F$ domestic at $T$:**
$$PV_{\text{pay}} = -K \cdot N_F \cdot P_D(t,T)$$

**Total domestic PV:**

$$\boxed{V_t^{(D)} = N_F \left( S_t \cdot P_F(t,T) - K \cdot P_D(t,T) \right)}$$

Andersen and Piterbarg provide the interpretation: the domestic value of one foreign zero-coupon bond is $X(t) \cdot P_F(t,T)$, where $X(t)$ is the spot FX rate. The forward simply combines a long foreign ZCB with a short domestic ZCB.

### 29.7.3 The Fair Forward and Zero-Value Condition

Define the *fair forward* (no-arbitrage forward rate):

$$F^*(t,T) = S_t \frac{P_F(t,T)}{P_D(t,T)}$$

Substituting into the PV formula:

$$V_t^{(D)} = N_F \cdot P_D(t,T) \left( F^*(t,T) - K \right)$$

This is the standard "discounted difference" form: **PV = notional × discount factor × (market forward − delivery price)**.

Hull confirms this structure with his general forward valuation result (equation 5.4): "the value of a forward contract at the time it is first entered into is close to zero... It is important for banks and other financial institutions to value the contract each day."

**Sanity check:** If $K = F^*$, then $V = 0$. The forward is struck at fair value at inception. ✓

> **Desk Reality: Marking FX Forwards**
>
> Traders mark forwards using the formula above. Key inputs:
> - **Spot rate:** Mid-market from a reliable source (Bloomberg, Reuters)
> - **Discount curves:** Domestic and foreign OIS curves (or SOFR/€STR equivalents)
> - **Delivery price K:** From the trade confirmation
>
> Common MTM discrepancies arise from:
> - Using different spot snapshots (different times of day)
> - Curve calibration differences (different instrument sets)
> - Day count mismatches in discount factor calculation
> - Holiday calendar differences affecting T
>
> Reconciliation with counterparties should identify and document curve differences upfront.

---

## 29.8 Hedging Foreign-Currency Cashflows

### 29.8.1 The Basic Hedge

Suppose you will receive a known foreign-currency amount $A_F$ at time $T$. Without hedging, your domestic value at $T$ is $S_T \cdot A_F$—exposed to FX risk.

**Hedge:** Enter a short forward (sell foreign forward) with $N_F = A_F$ and delivery price $K = F^*(0,T)$.

At maturity:
- You receive $A_F$ foreign from your underlying exposure
- You deliver $A_F$ into the forward, receiving $K \cdot A_F$ domestic

Your domestic payoff is $K \cdot A_F = F^*(0,T) \cdot A_F$, which is known today. The FX risk is eliminated.

Hull's ImportCo example illustrates exactly this: "ImportCo could hedge its foreign exchange risk by buying pounds from the financial institution in the 3-month forward market at 1.2225. This would have the effect of fixing the price to be paid to the British exporter at $12,225,000."

### 29.8.2 The Interest Differential Interpretation

When you hedge at the fair forward, you lock in the spot rate *adjusted* by the interest differential:

$$K = F^*(0,T) = S_0 \frac{P_F(0,T)}{P_D(0,T)}$$

If domestic rates exceed foreign rates, $P_D < P_F$, so $K > S_0$: you pay more per unit of foreign in the forward than at spot. This is the "cost" of hedging—but it's not a penalty. It exactly reflects that you could have earned higher domestic interest rather than foreign interest over the hedge period.

### 29.8.3 Hedging a Foreign Bond: The Return Transformation

A hedged foreign bond earns the **domestic** rate, not the foreign rate. This is not immediately obvious but follows directly from CIP.

**Example:**
- Hold a EUR zero-coupon bond paying €1,000,000 at $T = 1$ year
- EUR discount factor: $P_F = 0.9750$ (implying ~2.56% EUR rate)
- USD discount factor: $P_D = 0.9600$ (implying ~4.08% USD rate)
- Spot: $S_0 = 1.2000$ USD/EUR

**Unhedged:** Your USD value at maturity depends on the unknown future spot rate.

**Hedged:** Sell €1,000,000 forward at $F = 1.20 \times 0.9750/0.9600 = 1.21875$

At maturity, receive $\$1,218,750$. Your hedged USD return:

$$\frac{1,218,750}{1,200,000 \times 0.9750} = \frac{1,218,750}{1,170,000} = 1.0417$$

This is $1/0.96 = 4.17\%$—the USD rate, not the EUR rate.

**The hedge transforms your rate exposure along with your currency exposure.** You "converted" the EUR bond into a synthetic USD bond.

### 29.8.4 A Key Insight on Hedging

Hull makes an important observation about hedging outcomes: "This example illustrates a key aspect of hedging. The purpose of hedging is to reduce risk. There is no guarantee that the outcome with hedging will be better than the outcome without hedging."

The value of hedging is certainty, not expected return. The hedger gives up the chance of favorable spot movements in exchange for eliminating the risk of unfavorable ones.

---

## 29.9 Risk Sensitivities of an FX Forward

### 29.9.1 FX Delta (Spot Sensitivity)

Differentiating the PV formula with respect to spot $S_t$ (holding curves fixed):

$$\boxed{\frac{\partial V_t^{(D)}}{\partial S_t} = N_F \cdot P_F(t,T)}$$

Hull's result for forward delta on an asset with yield $q$ is $e^{-qT}$. For FX, $q = r_f$, so delta $= e^{-r_f T} = P_F$ under flat continuous rates.

Hull states explicitly: "For the delta of a forward foreign exchange contract, it is set equal to the foreign risk-free rate, $r_f$."

**Intuition:** The delta is the present value of the foreign notional. A one-unit move in spot changes the domestic value of receiving $N_F$ foreign by $N_F \cdot P_F$.

> **Desk Reality: Why Delta ≠ Notional**
>
> A common error is thinking an FX forward on €10mm has delta of €10mm. The correct delta is €10mm × $P_F$.
>
> For a 1-year forward with $P_F = 0.97$, delta is €9.7mm. This matters for:
> - Sizing spot hedges
> - VAR calculations
> - Aggregating positions across tenors
>
> The difference compounds at longer maturities. A 5-year forward at $P_F = 0.85$ has delta only 85% of notional.

### 29.9.2 Interest Rate Sensitivities

From the PV formula $V = N_F(S_t P_F - K P_D)$:

- **Foreign rate exposure:** The position is long $N_F \cdot S_t$ units of the foreign discount factor $P_F(t,T)$. If foreign rates rise, $P_F$ falls, reducing PV.

- **Domestic rate exposure:** The position is short $N_F \cdot K$ units of the domestic discount factor $P_D(t,T)$. If domestic rates rise, $P_D$ falls, but since the sign is negative, PV increases.

In multi-curve frameworks, the choice of which discount curves ($P_D$, $P_F$) correspond to which funding or collateral arrangement becomes critical—this is addressed in Chapter 21 (cross-currency curves) and Chapter 33 (collateral discounting).

### 29.9.3 Delta of Forward vs Delta of Spot Position

Hull notes an important subtlety: "The concept of delta can be applied to financial instruments other than options. Consider a forward contract on a non-dividend-paying stock. Equation (5.5) shows that the value of a forward contract is $S_0 - K e^{-rT}$... The delta of a long forward contract on one share is therefore always 1.0."

For FX forwards, the delta is $e^{-r_f T}$ rather than 1.0, because foreign currency pays a continuous "dividend" at rate $r_f$. This is consistent with treating foreign currency as an asset with known yield.

---

## 29.10 Non-Deliverable Forwards (NDFs)

> **Practitioner Note:** NDFs are not covered in Hull or Andersen-Piterbarg. This section is based on standard market practice and is marked as Category B (Claude-extended).

### 29.10.1 What Is an NDF?

A **non-deliverable forward** is an FX forward where, at maturity, there is no physical exchange of the two currencies. Instead, the contract settles in cash, typically in USD, based on the difference between the contracted forward rate and a reference "fixing" rate.

**Why NDFs exist:** Some currencies have capital controls or restrictions that prevent foreign investors from freely exchanging the currency. Rather than physically exchanging the restricted currency, parties settle the economic equivalent in a freely convertible currency.

### 29.10.2 Common NDF Currencies

| Currency | Why Restricted |
|----------|----------------|
| **BRL** (Brazilian Real) | Capital controls, IOF tax |
| **CNY/CNH** (Chinese Yuan) | Onshore controls; CNH is offshore but NDF market persists |
| **INR** (Indian Rupee) | Partial capital controls |
| **KRW** (Korean Won) | Foreign investor restrictions |
| **TWD** (Taiwan Dollar) | Limited onshore access |
| **IDR** (Indonesian Rupiah) | Capital controls |
| **PHP** (Philippine Peso) | Access restrictions |

### 29.10.3 NDF Mechanics

**At trade inception:**
- Agree on notional in foreign currency ($N_F$)
- Agree on forward rate ($K$)
- Agree on fixing source (e.g., central bank rate, EMTA fixing)
- Agree on settlement currency (usually USD)

**At maturity:**
1. The fixing rate $S_{\text{fix}}$ is determined from the agreed source
2. Calculate the settlement amount:

$$\boxed{\text{Settlement (USD)} = N_F \times \frac{S_{\text{fix}} - K}{S_{\text{fix}}}}$$

Wait—why divide by $S_{\text{fix}}$? Because the notional is in foreign currency, but settlement is in USD. We must convert the foreign-currency P&L to USD at the fixing rate.

**Alternative formula (equivalent):**

$$\text{Settlement (USD)} = N_F \times (S_{\text{fix}} - K) / S_{\text{fix}} = N_F / S_{\text{fix}} \times (S_{\text{fix}} - K)$$

where $N_F / S_{\text{fix}}$ converts the foreign notional to USD.

### 29.10.4 Worked NDF Example

**Setup:**
- Trade: Buy BRL 10,000,000 vs USD at forward rate K = 5.0000 (USD/BRL)
- At maturity: BRL fixing = 5.2500

**Settlement calculation:**
- You agreed to buy BRL at 5.00 (i.e., pay 1 USD to get 5 BRL)
- The fixing is 5.25 (BRL is weaker—1 USD now buys 5.25 BRL)
- You "won"—you locked in a better rate than the fixing

$$\text{Settlement} = 10{,}000{,}000 \times \frac{5.25 - 5.00}{5.25} = 10{,}000{,}000 \times 0.04762 = \text{USD } 476{,}190$$

You receive USD 476,190 from the counterparty.

**Sanity check:** The notional in USD terms is 10mm / 5.25 = USD 1.905mm. Your gain is (5.25 - 5.00) / 5.25 = 4.76% of that = USD 90,476. Wait, that doesn't match. Let me recalculate.

Actually: The correct formula should be $(K - S_{\text{fix}})$ or $(S_{\text{fix}} - K)$ depending on which direction you're long:

If you're buying BRL (selling USD), and BRL weakens (S goes up = more BRL per USD):
- You agreed to buy BRL at 5.00 (i.e., pay USD 2,000,000 to get BRL 10,000,000)
- At fixing, that same BRL 10,000,000 is worth only USD 1,904,762
- You overpaid by USD 95,238—you lose

Let me correct the example:

$$\text{Settlement (if buying BRL)} = N_F \times \left(\frac{1}{K} - \frac{1}{S_{\text{fix}}}\right)$$

$$= 10{,}000{,}000 \times \left(\frac{1}{5.00} - \frac{1}{5.25}\right) = 10{,}000{,}000 \times (0.20 - 0.1905) = -\text{USD } 95{,}238$$

You pay USD 95,238 to the counterparty.

### 29.10.5 Fixing Risk

The critical risk unique to NDFs is **fixing risk**: the reference rate on fixing day may not reflect the true market rate you could transact at.

Sources of fixing risk:
- **Illiquidity on fixing day:** Low volume can cause fixing to diverge from true tradeable rates
- **Official vs market rate:** Central bank fixings may be set administratively
- **Timing mismatch:** Fixing time may differ from your desired execution time
- **Manipulation risk:** Historical concerns about benchmark manipulation

> **Desk Reality: Managing NDF Fixing Risk**
>
> Sophisticated NDF traders manage fixing risk by:
> - Trading the underlying spot around the fixing window to "lock in" the rate
> - Using fixing options to protect against adverse fixing moves
> - Diversifying fixing sources where possible
> - Monitoring fixing-to-market deviations for early warning signs

---

## 29.11 P&L Attribution for FX Forwards

> **Practitioner Note:** P&L attribution for FX forwards is standard desk practice but not explicitly detailed in Hull or Andersen. This section is marked as Category B.

### 29.11.1 The Components of FX Forward P&L

Total P&L from an FX forward position decomposes into:

1. **Spot P&L:** Change in forward value due to spot rate movement
2. **Forward Point (Curve) P&L:** Change due to interest rate differential movement
3. **Time/Carry P&L:** Change from passage of time (forward converging to spot)
4. **Curve Shape P&L:** Changes in yield curve shape (beyond parallel shifts)

### 29.11.2 First-Order Decomposition

For small changes, the PV change decomposes as:

$$\Delta V \approx \underbrace{N_F \cdot P_F \cdot \Delta S}_{\text{Spot P\&L}} + \underbrace{N_F \cdot S \cdot \Delta P_F - N_F \cdot K \cdot \Delta P_D}_{\text{Rates P\&L}}$$

**Spot P&L** is intuitive: the position gains/loses based on spot movement times delta.

**Rates P&L** captures the impact of domestic and foreign rate changes on the discount factors.

### 29.11.3 Carry: The Time-Decay Component

Unlike options, forwards don't have time decay in the traditional sense. But they do have **carry**—the P&L from time passing if nothing else changes.

If you're long a forward at $K < F^*$ (profitable position), as time passes and the forward approaches maturity, the position value converges to $(F^* - K) \times N_F$. The rate of this convergence is the carry.

For a position struck at the fair forward ($K = F^*$), daily carry equals the forward point earn-out:

$$\text{Daily Carry} \approx N_F \times S \times (r - r_f) \times \frac{1}{365}$$

### 29.11.4 Worked P&L Attribution Example

**Position:** Long EUR 10mm vs USD, K = 1.0950, original forward = 1.0950 (struck at fair)

**Day 0:**
- Spot: 1.0800
- 3M Forward: 1.0950
- USD 3M rate: 5.5%
- EUR 3M rate: 4.0%
- PV = 0 (struck at fair forward)

**Day 30:**
- Spot: 1.0900 (+100 pips)
- 3M (now 2M) Forward: 1.0975
- USD rate: 5.6% (+10bp)
- EUR rate: 3.9% (-10bp)

**P&L Attribution:**

1. **Spot P&L:** $\Delta S \times N_F \times P_F \approx 0.01 \times 10{,}000{,}000 \times 0.99 = +\$99{,}000$

2. **Rates P&L:** The rate differential widened (USD up, EUR down), making EUR at a larger forward premium. This partially offsets or adds to spot P&L depending on the position. Estimate: additional ~$5,000.

3. **Carry P&L:** 30 days of carry at ~1.5% annualized differential = ~$12,000.

4. **Total P&L:** ~$116,000

(Exact attribution requires full revaluation at intermediate points.)

---

## 29.12 Rolling Positions: Tom/Next Swaps (Brief Introduction)

> **Practitioner Note:** This section provides a brief overview. Full treatment of FX swaps appears in Chapter 30.

### 29.12.1 Why Rolling Is Necessary

A spot FX position settles in T+2 days. If you want to maintain a position beyond settlement without taking physical delivery, you must "roll" it forward.

### 29.12.2 The Tom/Next Swap

A **Tom/Next (T/N) swap** is the most common rolling mechanism:

- **Near leg:** Sell (or buy) foreign currency for value "tomorrow" (T+1)
- **Far leg:** Buy (or sell) the same amount for value "next" (T+2, i.e., spot)

The difference between the two rates is the **Tom/Next points**, which reflect one day's worth of interest differential.

### 29.12.3 Implied Overnight Rate

From T/N points, you can derive the implied overnight funding rate:

$$\text{Implied Rate} = \frac{\text{T/N Points}}{\text{Spot}} \times \frac{360}{1} \times 100$$

This should approximately equal the overnight interest differential. Deviations indicate funding stress or year-end effects.

### 29.12.4 Connection to Cross-Currency Basis

Persistent deviations between T/N implied rates and actual OIS rates reflect **cross-currency basis**—the topic of Chapter 30. When dollar funding is scarce, T/N points widen beyond what pure interest differentials would suggest.

**Full treatment of FX swaps, cross-currency swaps, and the cross-currency basis appears in Chapter 30.**

---

## 29.13 Worked Examples

> **Note:** Numbers are illustrative. Settlement conventions (T+2, T+1, etc.) and day-count conventions must be adapted to specific currency pairs in practice.

### Example A: Computing the Forward from Rates

**Given:**
- Spot: $S_0^{\text{USD/EUR}} = 1.2000$ (1.20 USD per EUR)
- USD rate: $r = 5\%$ continuous
- EUR rate: $r_f = 2\%$ continuous
- Maturity: $T = 0.5$ years

**Compute forward:**

$$F_0 = S_0 e^{(r - r_f)T} = 1.2000 \times e^{(0.05 - 0.02) \times 0.5} = 1.2000 \times e^{0.015}$$

$$e^{0.015} \approx 1.0151$$

$$F_0 \approx 1.2181 \text{ USD/EUR}$$

**Forward points:** $F_0 - S_0 = 1.2181 - 1.2000 = 0.0181$ USD/EUR = 181 pips

**Interpretation:** USD rates exceed EUR rates, so EUR trades at a forward premium (more dollars per euro in the forward than at spot).

### Example B: Forward from Discount Factors

**Given:**
- Spot: $S_0 = 1.2000$ USD/EUR
- 3-month discount factors (simple rates $R_D = 4\%$, $R_F = 1\%$, $T = 0.25$):
  - $P_D = 1/(1 + 0.04 \times 0.25) = 1/1.01 = 0.9901$
  - $P_F = 1/(1 + 0.01 \times 0.25) = 1/1.0025 = 0.9975$

**Compute forward:**

$$F_0 = S_0 \frac{P_F}{P_D} = 1.2000 \times \frac{0.9975}{0.9901} = 1.2000 \times 1.00747 = 1.2090$$

**Forward points:** 90 pips

**Verification via rate form:**

$$F_0 = S_0 \frac{1 + R_D T}{1 + R_F T} = 1.2000 \times \frac{1.01}{1.0025} = 1.2090 \checkmark$$

The DF form and rate form give identical answers under consistent conventions.

### Example C: PV of an Off-Market Forward

**Given:**
- Notional: $N_F = 10,000,000$ EUR
- Spot: $S_0 = 1.2000$ USD/EUR
- Discount factors (from Example B): $P_D = 0.9901$, $P_F = 0.9975$
- Fair forward: $F^* = 1.2090$ USD/EUR
- Trade forward (delivery price): $K = 1.2100$ USD/EUR (above fair)

**Compute PV (long forward = buy EUR):**

$$V_0^{(\text{USD})} = N_F \left( S_0 P_F - K P_D \right)$$

$$= 10{,}000{,}000 \times \left( 1.2000 \times 0.9975 - 1.2100 \times 0.9901 \right)$$

$$= 10{,}000{,}000 \times \left( 1.1970 - 1.1980 \right) = 10{,}000{,}000 \times (-0.0010) = -10{,}000 \text{ USD}$$

**Interpretation:** The long forward (buy EUR) agreed to pay 1.21 USD/EUR when the fair forward is only 1.209. The forward is underwater by about $10,000.

**Alternative calculation:** $V = N_F \cdot P_D \cdot (F^* - K) = 10M \times 0.9901 \times (1.2090 - 1.2100) = -9{,}901$ USD (small difference due to rounding).

### Example D: Arbitrage When Forward Is Mispriced (Hull-Style)

**Given:** Same as Example B, but market forward is $F_{\text{mkt}} = 1.2200$ (too high vs CIP-implied 1.2090).

**Arbitrage strategy:**

1. **Borrow USD:** $\$1.2000$ at 4% for 0.25 years → repay $\$1.2000 \times 1.01 = \$1.212$

2. **Buy EUR at spot:** $\$1.20$ buys €1.00

3. **Invest EUR:** €1.00 at 1% for 0.25 years → receive €$1.0025$

4. **Sell EUR forward:** Deliver €1.0025 at $F_{\text{mkt}} = 1.2200$ → receive $\$1.2231$

**Net at maturity:**
- Receive: $\$1.2231$
- Repay: $\$1.2120$
- **Profit: $\$0.0111$ per €1 initial**

Scaled to €10,000,000: profit = $\$111,000$.

This is risk-free profit, so arbitrageurs would trade until $F_{\text{mkt}} = F^*$.

### Example E: NDF Settlement

**Given:**
- NDF to sell KRW 1,000,000,000 vs USD
- Contracted forward rate: K = 1,320 KRW/USD
- Fixing rate at maturity: S = 1,350 KRW/USD (KRW weakened)

**Settlement calculation:**

You agreed to sell KRW (buy USD) at 1,320. The fixing is 1,350 (KRW is weaker).
- Your USD proceeds at contracted rate: KRW 1bn / 1,320 = USD 757,576
- Fair USD value at fixing: KRW 1bn / 1,350 = USD 740,741
- You "won" by USD 16,835 (you locked in a stronger KRW)

$$\text{Settlement} = N_{\text{KRW}} \times \left(\frac{1}{K} - \frac{1}{S_{\text{fix}}}\right) = 1{,}000{,}000{,}000 \times \left(\frac{1}{1320} - \frac{1}{1350}\right)$$

$$= 1{,}000{,}000{,}000 \times (0.0007576 - 0.0007407) = +\text{USD } 16{,}835$$

You receive USD 16,835.

---

## 29.14 Practical Notes and Common Pitfalls

### Quote Direction Errors

The most common implementation error is inverting the quote without inverting the DF ratio:

$$F^{F/D} = S^{F/D} \times \frac{P_D}{P_F}$$

Note the ratio flips when the quote direction flips. Systems that store currency pairs inconsistently will produce wrong hedge ratios.

### Compounding Convention Mismatch

If you use continuous-rate CIP ($F = S e^{(r - r_f)T}$) but plug in simple rates, you get spurious "arbitrage" signals.

**Example:** With $T = 2$ years, simple rates $R_D = 10\%$, $R_F = 2\%$:
- Correct (simple): $F = 1.2 \times (1.20/1.04) = 1.385$
- Wrong (treating as continuous): $F = 1.2 \times e^{0.16} = 1.408$

The 2.3-cent difference is not arbitrage—it's convention mismatch. **Always use discount factors to avoid this trap.**

### Settlement Timing

The forward settlement date determines $T$. If the market defines "spot" as settling T+2, ensure the forward's time-to-maturity is measured consistently.

### Bid-Ask Spreads

Forward points have bid-ask spreads; so does spot. The no-arbitrage band has width proportional to transaction costs. CIP holds within the band; observed deviations are often within bid-ask.

### FX Forward Liquidity

Andersen and Piterbarg note that "the interbank FX forward market is rarely liquid beyond maturities of one year." For longer hedging horizons, market participants typically use cross-currency basis swaps (Chapter 30) rather than FX forwards.

### Verification Checklist

| Test | What to Check |
|------|---------------|
| **Dimension/units** | $S$, $F$, $K$ all in same units ($D/F$). $P_D$, $P_F$ dimensionless. |
| **CIP replication** | Reproduce $F$ from borrow/lend/spot; confirm no free profit. |
| **Zero-value test** | $V(K = F^*) = 0$ when curves are consistent. |
| **Sign sanity** | Higher domestic rates → higher $F$ (in $D/F$ quotes). |
| **Delta check** | FX delta = $N_F \cdot P_F$, not $N_F$. |
| **Settlement date** | Forward tenor calculated from correct spot date. |

---

## Summary

The FX forward market is not a forecasting market—it's an interest rate market in disguise. Every forward rate is pinned down by the spot rate and the interest differential across currencies. Understanding this is essential for anyone managing cross-currency exposures.

**Key takeaways:**

1. **CIP determines the forward:** $F = S \times P_F / P_D$ links FX forwards to discount curves.

2. **Forward points = interest differential:** The forward-to-spot ratio reflects relative funding costs, not currency forecasts.

3. **Forward ≠ expected future spot:** This is the fundamental misconception. The forward is arbitrage pricing, not a prediction.

4. **Foreign currency = asset with yield:** Treat foreign currency as earning the foreign risk-free rate; this makes CIP a special case of dividend-adjusted forward pricing.

5. **Hedging transforms rate exposure:** A hedged foreign bond earns the domestic rate, not the foreign rate—the hedge "converts" the rate exposure along with the currency exposure.

6. **PV formula:** $V = N_F(S P_F - K P_D)$, equivalently $V = N_F P_D(F^* - K)$.

7. **FX delta:** $\partial V / \partial S = N_F P_F$—the present value of the foreign leg.

8. **Quote direction matters:** Inverting the quote inverts the DF ratio. Mixing conventions creates spurious signals.

9. **NDFs settle in cash:** For restricted currencies, no physical exchange occurs; settlement is based on a fixing rate.

10. **Liquidity beyond 1 year:** Andersen and Piterbarg note FX forwards are "rarely liquid beyond maturities of one year"—longer hedges use cross-currency basis swaps (Chapter 30).

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| Spot FX rate $S^{D/F}$ | Domestic currency per 1 foreign | Foundation for all FX calculations; quote direction critical |
| CIP (continuous form) | $F = S e^{(r - r_f)T}$ | No-arbitrage link between spot, forward, and rates |
| CIP (DF form) | $F = S \cdot P_F / P_D$ | Convention-robust; avoids compounding ambiguity |
| Forward points | $F - S$ | Reflects interest differential, not forecast |
| Forward premium/discount | $F > S$ or $F < S$ | Higher domestic rates → forward premium |
| FX forward PV | $N_F(S P_F - K P_D)$ | Discount each currency leg, convert to domestic |
| Fair forward | $F^* = S \cdot P_F / P_D$ | Delivery price that makes PV = 0 at inception |
| FX delta | $N_F \cdot P_F$ | Spot sensitivity of forward position |
| NDF | Cash-settled FX forward | Used for restricted currencies |
| Fixing risk | Risk that fixing differs from market | Unique to NDFs; managed via hedging/diversification |
| Value date | Settlement date for FX | T+2 for most; T+1 for USD/CAD |

---

## Notation for This Chapter

| Symbol | Definition |
|--------|------------|
| $D$, $F$ | Domestic and foreign currencies |
| $S_t^{D/F}$ | Spot FX at time $t$ (domestic per 1 foreign) |
| $F(t,T)$ | Forward FX rate for maturity $T$ |
| $K$ | Delivery price (strike) of FX forward |
| $P_D(t,T)$ | Domestic discount factor |
| $P_F(t,T)$ | Foreign discount factor |
| $N_F$ | Foreign currency notional |
| $r$, $r_f$ | Domestic and foreign risk-free rates |
| $S_{\text{fix}}$ | NDF fixing rate |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What does $S_t^{D/F}$ represent? | Domestic currency units per 1 foreign unit |
| 2 | State CIP in discount factor form. | $F = S \cdot P_F / P_D$ |
| 3 | State CIP in continuous rate form. | $F = S \cdot e^{(r - r_f)T}$ |
| 4 | Why can foreign currency be treated as an asset with yield? | Holding foreign currency earns the foreign risk-free rate |
| 5 | If domestic rates > foreign rates, is the forward at premium or discount? | Premium ($F > S$ in $D/F$ quotes) |
| 6 | What are "forward points"? | $F - S$; the forward-to-spot difference |
| 7 | What drives forward points? | The interest differential between currencies |
| 8 | What is the PV formula for a long FX forward? | $V = N_F(S \cdot P_F - K \cdot P_D)$ |
| 9 | When is an FX forward worth zero at inception? | When $K = F^*$ (struck at the fair forward) |
| 10 | What is the FX delta of a forward? | $N_F \cdot P_F$ |
| 11 | How do you hedge a known future foreign cash inflow? | Short forward (sell foreign forward) with $N_F =$ inflow amount |
| 12 | What return does a hedged foreign ZCB earn? | Approximately the domestic rate, not the foreign rate |
| 13 | What happens to CIP formulas when you invert the quote? | The DF ratio inverts: $F^{F/D} = S^{F/D} \cdot P_D / P_F$ |
| 14 | Why is the DF form of CIP preferred? | It's convention-robust; avoids compounding confusion |
| 15 | What is "covered" about covered interest parity? | The FX conversion at maturity is locked in via a forward |
| 16 | Does the forward rate predict future spot rates? | No; it reflects current interest differentials, not expectations |
| 17 | What is an NDF? | A cash-settled FX forward for restricted currencies |
| 18 | What is fixing risk in NDFs? | Risk that the official fixing rate diverges from market rates |
| 19 | What settlement convention do most FX pairs use? | T+2 (two business days after trade date) |
| 20 | What is a pip in FX? | 0.0001 for most pairs; 0.01 for JPY pairs |
| 21 | What creates apparent arbitrage signals in CIP calculations? | Mixing compounding conventions (simple vs continuous) |
| 22 | How do bid-ask spreads affect CIP? | They widen the no-arbitrage band; small deviations are normal |
| 23 | What is the "forward rate bias" puzzle? | High-rate currencies tend to appreciate, contrary to forward prediction |
| 24 | What is a Tom/Next swap used for? | Rolling spot positions to avoid physical delivery |
| 25 | What does Andersen & Piterbarg say about FX forward liquidity? | Rarely liquid beyond ~1 year; longer horizons use xccy swaps |

---

## Mini Problem Set

### Problems

1. Given $S^{\text{USD/EUR}} = 1.2500$, compute $S^{\text{EUR/USD}}$.

2. Show that inverting the quote inverts the DF ratio: if $F^{D/F} = S^{D/F} \cdot P_F/P_D$, derive $F^{F/D}$.

3. With $S_0 = 1.10$ USD/EUR, $P_D(0, 0.5) = 0.98$, $P_F(0, 0.5) = 0.995$, compute $F(0, 0.5)$.

4. Compute PV (in USD) of a long forward with $N_F = 5{,}000{,}000$ EUR, $S_0 = 1.10$, $K = 1.12$, $P_D = 0.99$, $P_F = 0.995$.

5. Explain (in words) the arbitrage strategy when $F_{\text{mkt}} > F^*$.

6. Using simple rates $R_D$, $R_F$, derive $F = S(1 + R_D T)/(1 + R_F T)$ from first principles.

7. If USD rates rise (all else equal), what happens to the USD/EUR forward?

8. A U.S. investor will receive €10 million in 6 months. What forward position hedges the FX risk?

9. Re-do Problem 4 using the formula $V = N_F \cdot P_D \cdot (F^* - K)$; verify it matches.

10. Compute the FX delta of the position in Problem 4.

11. A trader says "the forward is predicting EUR appreciation." Explain why this interpretation is wrong.

12. An NDF on KRW 500,000,000 is contracted at 1,300 KRW/USD. At fixing, the rate is 1,280. Calculate the USD settlement amount and determine who pays whom.

### Solution Sketches

**Problem 1:**
$$S^{\text{EUR/USD}} = 1/1.2500 = 0.8000$$

**Problem 2:**
Start with $F^{D/F} = S^{D/F} \cdot P_F/P_D$. Taking inverse:
$$F^{F/D} = \frac{1}{F^{D/F}} = \frac{1}{S^{D/F}} \cdot \frac{P_D}{P_F} = S^{F/D} \cdot \frac{P_D}{P_F}$$

**Problem 3:**
$$F(0, 0.5) = 1.10 \times \frac{0.995}{0.98} = 1.10 \times 1.0153 = 1.1168$$

**Problem 4:**
$$V = 5{,}000{,}000 \times (1.10 \times 0.995 - 1.12 \times 0.99)$$
$$= 5{,}000{,}000 \times (1.0945 - 1.1088) = 5{,}000{,}000 \times (-0.0143) = -71{,}500 \text{ USD}$$

**Problem 5:**
If $F_{\text{mkt}} > F^*$: borrow domestic, convert to foreign at spot, invest foreign, sell foreign forward at the high rate. At maturity, the forward proceeds exceed the domestic borrowing cost, locking in risk-free profit.

**Problem 9:**
$F^* = 1.10 \times 0.995/0.99 = 1.1056$
$V = 5M \times 0.99 \times (1.1056 - 1.12) = 5M \times 0.99 \times (-0.0144) = -71{,}280$ USD
Matches within rounding. ✓

**Problem 10:**
FX delta = $N_F \times P_F = 5{,}000{,}000 \times 0.995 = 4{,}975{,}000$ EUR-equivalent sensitivity.

**Problem 11:**
The forward reflects the interest differential via CIP, not market expectations. Higher USD rates than EUR rates cause EUR to trade at a forward premium—this is mechanical arbitrage pricing, not a forecast.

**Problem 12:**
If selling KRW (buying USD) at K=1300, and fixing is 1280 (KRW stronger):
$$\text{Settlement} = 500{,}000{,}000 \times \left(\frac{1}{1300} - \frac{1}{1280}\right)$$
$$= 500{,}000{,}000 \times (0.0007692 - 0.0007813) = -\text{USD } 6{,}010$$
You pay USD 6,010 (you locked in a weaker KRW rate; actual KRW is stronger).

---

## References

- Hull, *Options, Futures, and Other Derivatives* (FX forward pricing; covered interest parity; hedging examples)
- Andersen & Piterbarg, *Interest Rate Modeling* (FX forwards and cross-currency arbitrage constraints)

*Cross-references: Ch 2 (discount factors), Ch 3 (zero/forward rates), Ch 21 (cross-currency curves), Ch 30 (FX swaps and cross-currency swaps), Ch 31 (multi-currency risk)*
