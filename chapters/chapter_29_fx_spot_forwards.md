# Chapter 29: FX Spot and Forwards — Pricing via Interest Differentials

---

## Introduction

EUR/USD spot is 1.10. The 1-year forward is 1.12. Is the market predicting that the euro will appreciate against the dollar?

The intuitive answer—"yes, the forward is higher, so the market expects EUR to rise"—is wrong. The forward rate has almost nothing to do with market expectations of future spot rates. Instead, the forward is pinned down by a fundamental no-arbitrage relationship: **covered interest parity (CIP)**. The forward differs from spot because interest rates differ across currencies. If you could earn more by investing in dollars than in euros, that advantage must be exactly offset by the forward FX rate—otherwise, you could lock in a risk-free profit.

This insight has profound implications for anyone managing cross-currency exposures. When a U.S. investor hedges a European bond with FX forwards, they are not betting on currency direction—they are locking in the interest differential. When a corporation hedges a future foreign-currency payable, the "cost" of hedging is not some penalty; it *is* the interest differential, which reflects the opportunity cost of holding one currency versus another. Understanding FX forwards means understanding that currency markets and interest rate markets are inextricably linked.

Hull presents a vivid example of corporate hedging: "ImportCo, a company based in the United States, knows that it will have to pay £10 million on August 21, 2020, for goods it has purchased from a British supplier... ImportCo could hedge its foreign exchange risk by buying pounds from the financial institution in the 3-month forward market at 1.2225. This would have the effect of fixing the price to be paid to the British exporter at $12,225,000." The forward eliminates uncertainty—but at what "cost"? The answer lies in understanding CIP.

This chapter covers:

1. **FX spot and quote conventions** (Section 29.1) — what $S^{D/F}$ means and why quote direction matters
2. **The FX forward contract** (Section 29.2) — structure and payoff at maturity
3. **Covered interest parity** (Section 29.3) — deriving $F = S \cdot P_F / P_D$ from no-arbitrage
4. **Forward points and interpretation** (Section 29.4) — the mechanics of $F - S$
5. **Carry and carry trades** (Section 29.5) — why the forward embeds the interest differential
6. **Valuing an FX forward** (Section 29.6) — PV in domestic currency terms
7. **Hedging with FX forwards** (Section 29.7) — locking in domestic value of foreign cashflows
8. **Risk sensitivities** (Section 29.8) — FX delta and rates exposure

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

### 29.1.3 Spot Settlement Conventions

**I'm not sure** about the precise spot settlement conventions from the provided sources. In practice, most major currency pairs settle T+2 (two business days after trade date), with some exceptions (e.g., USD/CAD settles T+1). The "spot rate" in valuation formulas should correspond to the exchange rate for settlement consistent with the forward's settlement timing. When precision matters—particularly for very short-dated forwards—the settlement lag must be accounted for in defining the time parameter $T$.

---

## 29.2 The FX Forward Contract

### 29.2.1 Contract Structure

A forward contract is an agreement made today to exchange assets at a future time $T$. An FX forward specifically exchanges two currencies:

- **Long FX forward (buy foreign):** At maturity $T$, receive $N_F$ units of foreign currency and pay $K \cdot N_F$ units of domestic currency, where $K$ is the delivery price (strike) agreed at inception.

Hull illustrates the corporate motivation clearly: "ImportCo could hedge its foreign exchange risk by buying pounds from the financial institution in the 3-month forward market at 1.2225. This would have the effect of fixing the price to be paid to the British exporter at $12,225,000."

The key feature is that $K$ is fixed at trade inception, while the spot rate $S_T$ at maturity is unknown. The forward eliminates the uncertainty about how many domestic dollars will be needed.

> **Analogy: The Storage Locker**
>
> Why is the Forward Rate different from the Spot Rate? Is it a prediction? No.
> Think of buying a Forward as **putting money in a foreign storage locker**.
>
> 1.  **Spot**: You exchange USD for EUR today.
> 2.  **Storage**: You put the EUR in a bank account (a "locker") for 1 year.
> 3.  **Interest**: The locker *pays* you interest (the EUR rate).
> 4.  **Forward**: The Forward Rate is just the price that accounts for the fact that you own a "fuller locker" at the end of the year.
>
> Buying a Forward is mathematically identical to buying Spot and earning interest. Therefore, the price *must* reflect that interest difference.

### 29.2.2 Payoff at Maturity

If you translate the payoff into domestic currency units at maturity:

$$\boxed{\text{Domestic-equivalent payoff at } T = N_F(S_T - K)}$$

This is the standard forward payoff: you agreed to buy foreign currency at $K$, but the spot rate at maturity is $S_T$. If $S_T > K$, the forward is profitable (you bought cheap); if $S_T < K$, you overpaid relative to spot.

**Unit check:** Both $S_T$ and $K$ are in $D/F$ units, so $(S_T - K) \cdot N_F$ is in domestic currency. ✓

Hull notes the parallel for the counterparty: "Consider next another U.S. company, which we will refer to as ExportCo, that is exporting goods to the United Kingdom and, on May 21, 2020, knows that it will receive £30 million 3 months later. ExportCo can hedge its foreign exchange risk by selling £30 million in the 3-month forward market." ExportCo takes the opposite position—short the forward (sell foreign forward)—and locks in its dollar proceeds.

---

## 29.3 Covered Interest Parity: The Core Pricing Relationship

### 29.3.1 The No-Arbitrage Argument

Hull presents the fundamental pricing relationship for currency forwards in Chapter 5:

$$\boxed{F_0 = S_0 e^{(r - r_f)T}}$$

where $r$ is the domestic risk-free rate and $r_f$ is the foreign risk-free rate (both continuously compounded).

The intuition comes from treating foreign currency as an asset providing a yield. As Hull explains: "A foreign currency has the property that the holder of the currency can earn interest at the risk-free interest rate prevailing in the foreign country. For example, the holder can invest the currency in a foreign-denominated bond." Thus, "the foreign currency can be regarded as an investment asset paying a known yield. The yield is the risk-free rate of interest in the foreign currency."

This is a powerful insight: **FX forward pricing is just dividend-adjusted forward pricing**, with the foreign rate playing the role of the dividend yield.

### 29.3.2 The Replication Argument

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

### 29.3.3 Worked Arbitrage Example (from Hull)

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

### 29.3.4 The Discount Factor Form

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

## 29.4 Forward Points: What They Reveal

### 29.4.1 Definition and Mechanics

Forward points are the difference between the forward rate and the spot rate:

$$\text{Forward points} = F(t,T) - S_t$$

Using the CIP formula:

$$F(t,T) - S_t = S_t \left( \frac{P_F(t,T)}{P_D(t,T)} - 1 \right)$$

This shows that forward points are driven entirely by the ratio of discount factors—which reflects the interest rate differential between the two currencies.

### 29.4.2 Premium vs Discount

- **Forward premium (in $D/F$ quotes):** If $F > S$, the foreign currency trades at a forward premium. This occurs when $P_F / P_D > 1$, meaning domestic rates are higher than foreign rates (equivalently, $P_D < P_F$ for the same maturity).

- **Forward discount (in $D/F$ quotes):** If $F < S$, the foreign currency trades at a forward discount. This occurs when foreign rates exceed domestic rates.

Hull provides market evidence: "For all the currencies considered in the table, short-term interest rates were lower than on the U.S. dollar. This corresponds to the $r > r_f$ situation and explains why the settlement futures prices of these currencies increase with maturity."

> **Intuition:** If you can earn more interest holding dollars than euros, then locking in a future EUR purchase via a forward must cost you something—you give up the higher domestic interest. That cost shows up as a forward premium on EUR (you pay more USD per EUR in the forward than at spot).

> **Rule of Thumb: The Forward Point Check**
>
> *   **High Yielding Currency** = Trades at a **Forward Discount**.
>     *   (You get *less* of it in the future per unit of base currency? No, wait. Price goes down.)
>     *   Price of Foreign Ccy (in Domestic terms) will be LOWER in forward.
>     *   *Why?* You get paid interest to hold it. So the market charges you a lower price to buy it future to offset the "free money" you'd get from holding it spot.
>
> *   **Low Yielding Currency** = Trades at a **Forward Premium**.
>     *   Price will be HIGHER in forward.
>     *   *Why?* You pay a penalty (negative carry) to hold it vs high-rate domestic. The forward executes at a higher price to compensate for the interest you *didn't* earn.
>
> **Visual: The Conveyor Belt**
> Imagine Spot is at the start of a conveyor belt.
> *   The **Motor** is the Interest Rate Differential.
> *   If Foreign Rate > Domestic Rate, the motor pulls the price **Down** (Discount).
> *   If Foreign Rate < Domestic Rate, the motor pushes the price **Up** (Premium).

### 29.4.3 Quote Inversion and Sign Conventions

Hull cautions that "conventions can differ" in how basis and forward points are expressed. A critical point: **inverting the quote flips the sign interpretation.**

If $F^{D/F} > S^{D/F}$ (forward premium in domestic-per-foreign terms), then:

$$F^{F/D} = \frac{1}{F^{D/F}} < \frac{1}{S^{D/F}} = S^{F/D}$$

In the inverted quote, the forward is at a *discount*. Same economics, different sign—depending on which way you read the quote.

**I'm not sure** about specific market conventions for "pips" or "points" scaling (e.g., some pairs quote points as the last two decimal places). These are pair-specific and must be verified with desk standards.

---

## 29.5 Carry and the Forward Embedded Cost

### 29.5.1 What "Carry" Means in FX

Traders often describe the forward as "spot plus carry." In fixed income terms, *carry* is the return from holding a position assuming prices don't move—the accrual from yield and funding differentials.

For FX forwards, carry is the interest differential embedded in $F/S$. Rewriting CIP:

$$\frac{F_0}{S_0} = e^{(r - r_f)T} \approx 1 + (r - r_f)T$$

The forward-to-spot ratio equals (approximately) one plus the interest differential times time. If domestic rates exceed foreign rates by 200 basis points annually, then over one year:

$$F_0 \approx S_0 \times 1.02$$

The forward is about 2% above spot.

Taleb, in *Dynamic Hedging*, notes that "currency carry corresponds to the overnight rate differential," emphasizing that carry in FX markets directly reflects the funding cost difference between the two currencies.

### 29.5.2 The "Carry Trade" and Forward Pricing

The classic "carry trade" involves borrowing in a low-interest-rate currency and investing in a high-interest-rate currency. The forward pricing relationship says that this carry advantage is *exactly offset* by the forward exchange rate—at least in a world where CIP holds perfectly.

**Example:** If USD rates are 5% and EUR rates are 2%, the forward market prices EUR at a 3% premium relative to spot. A carry trader who borrows EUR and invests in USD earns 3% in interest differential—but if they hedge the currency risk, they give up exactly 3% through the forward. The net is zero (before bid-ask spreads).

This is the essence of CIP: there is no free lunch from covered interest arbitrage. The forward embeds the interest differential, not market expectations of currency direction.

> **Why this matters for hedging:** When a domestic investor hedges a foreign bond, the "cost" of hedging is not some penalty—it *is* the interest differential, which the investor receives through a lower foreign yield. The hedged return should approximately equal the domestic rate (see worked example in Section 29.9).

### 29.5.3 Forward Rates and Exchange Rate Forecasts

Hull poses an important question: "It is sometimes argued that a forward exchange rate is an unbiased predictor of future exchange rates. Under what circumstances is this so?"

The answer relates to the distinction between covered and *uncovered* interest parity. CIP is a no-arbitrage relationship that must hold (within transaction costs). Uncovered interest parity (UIP) is an economic hypothesis about expected future spot rates that empirically often fails—the famous "forward premium puzzle." This chapter focuses on CIP; UIP is a separate topic.

---

## 29.6 Valuing an FX Forward: PV in Domestic Currency

### 29.6.1 The Two-Leg Decomposition

An FX forward has two cashflows at maturity:
- Receive $N_F$ units of foreign currency
- Pay $K \cdot N_F$ units of domestic currency

Each leg is valued in domestic currency, then netted.

### 29.6.2 Domestic PV Formula

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

### 29.6.3 The Fair Forward and Zero-Value Condition

Define the *fair forward* (no-arbitrage forward rate):

$$F^*(t,T) = S_t \frac{P_F(t,T)}{P_D(t,T)}$$

Substituting into the PV formula:

$$V_t^{(D)} = N_F \cdot P_D(t,T) \left( F^*(t,T) - K \right)$$

This is the standard "discounted difference" form: **PV = notional × discount factor × (market forward − delivery price)**.

Hull confirms this structure with his general forward valuation result (equation 5.4): "the value of a forward contract at the time it is first entered into is close to zero... It is important for banks and other financial institutions to value the contract each day."

**Sanity check:** If $K = F^*$, then $V = 0$. The forward is struck at fair value at inception. ✓

---

## 29.7 Hedging Foreign-Currency Cashflows

### 29.7.1 The Basic Hedge

Suppose you will receive a known foreign-currency amount $A_F$ at time $T$. Without hedging, your domestic value at $T$ is $S_T \cdot A_F$—exposed to FX risk.

**Hedge:** Enter a short forward (sell foreign forward) with $N_F = A_F$ and delivery price $K = F^*(0,T)$.

At maturity:
- You receive $A_F$ foreign from your underlying exposure
- You deliver $A_F$ into the forward, receiving $K \cdot A_F$ domestic

Your domestic payoff is $K \cdot A_F = F^*(0,T) \cdot A_F$, which is known today. The FX risk is eliminated.

Hull's ImportCo example illustrates exactly this: "ImportCo could hedge its foreign exchange risk by buying pounds from the financial institution in the 3-month forward market at 1.2225. This would have the effect of fixing the price to be paid to the British exporter at $12,225,000."

### 29.7.2 The Interest Differential Interpretation

When you hedge at the fair forward, you lock in the spot rate *adjusted* by the interest differential:

$$K = F^*(0,T) = S_0 \frac{P_F(0,T)}{P_D(0,T)}$$

If domestic rates exceed foreign rates, $P_D < P_F$, so $K > S_0$: you pay more per unit of foreign in the forward than at spot. This is the "cost" of hedging—but it's not a penalty. It exactly reflects that you could have earned higher domestic interest rather than foreign interest over the hedge period.

### 29.7.3 A Key Insight on Hedging

Hull makes an important observation about hedging outcomes: "This example illustrates a key aspect of hedging. The purpose of hedging is to reduce risk. There is no guarantee that the outcome with hedging will be better than the outcome without hedging."

The value of hedging is certainty, not expected return. The hedger gives up the chance of favorable spot movements in exchange for eliminating the risk of unfavorable ones.

---

## 29.8 Risk Sensitivities of an FX Forward

### 29.8.1 FX Delta (Spot Sensitivity)

Differentiating the PV formula with respect to spot $S_t$ (holding curves fixed):

$$\boxed{\frac{\partial V_t^{(D)}}{\partial S_t} = N_F \cdot P_F(t,T)}$$

Hull's result for forward delta on an asset with yield $q$ is $e^{-qT}$. For FX, $q = r_f$, so delta $= e^{-r_f T} = P_F$ under flat continuous rates.

Hull states explicitly: "For the delta of a forward foreign exchange contract, it is set equal to the foreign risk-free rate, $r_f$."

**Intuition:** The delta is the present value of the foreign notional. A one-unit move in spot changes the domestic value of receiving $N_F$ foreign by $N_F \cdot P_F$.

### 29.8.2 Interest Rate Sensitivities

From the PV formula $V = N_F(S_t P_F - K P_D)$:

- **Foreign rate exposure:** The position is long $N_F \cdot S_t$ units of the foreign discount factor $P_F(t,T)$. If foreign rates rise, $P_F$ falls, reducing PV.

- **Domestic rate exposure:** The position is short $N_F \cdot K$ units of the domestic discount factor $P_D(t,T)$. If domestic rates rise, $P_D$ falls, but since the sign is negative, PV increases.

In multi-curve frameworks, the choice of which discount curves ($P_D$, $P_F$) correspond to which funding or collateral arrangement becomes critical—this is addressed in Chapter 21 (cross-currency curves) and Chapter 33 (collateral discounting).

### 29.8.3 Delta of Forward vs Delta of Spot Position

Hull notes an important subtlety: "The concept of delta can be applied to financial instruments other than options. Consider a forward contract on a non-dividend-paying stock. Equation (5.5) shows that the value of a forward contract is $S_0 - K e^{-rT}$... The delta of a long forward contract on one share is therefore always 1.0."

For FX forwards, the delta is $e^{-r_f T}$ rather than 1.0, because foreign currency pays a continuous "dividend" at rate $r_f$. This is consistent with treating foreign currency as an asset with known yield.

---

## 29.9 Worked Examples

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

**Forward points:** $F_0 - S_0 = 1.2181 - 1.2000 = 0.0181$ USD/EUR

**Interpretation:** USD rates exceed EUR rates, so EUR trades at a forward premium (more dollars per euro in the forward than at spot).

### Example B: Forward from Discount Factors

**Given:**
- Spot: $S_0 = 1.2000$ USD/EUR
- 3-month discount factors (simple rates $R_D = 4\%$, $R_F = 1\%$, $T = 0.25$):
  - $P_D = 1/(1 + 0.04 \times 0.25) = 1/1.01 = 0.9901$
  - $P_F = 1/(1 + 0.01 \times 0.25) = 1/1.0025 = 0.9975$

**Compute forward:**

$$F_0 = S_0 \frac{P_F}{P_D} = 1.2000 \times \frac{0.9975}{0.9901} = 1.2000 \times 1.00747 = 1.2090$$

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

### Example E: Hedging a Foreign Bond

**Given:**
- Hold a EUR zero-coupon bond paying €1,000,000 at $T = 1$ year
- $P_F(0,1) = 0.9750$ (EUR discount factor)
- $P_D(0,1) = 0.9600$ (USD discount factor)
- Spot: $S_0 = 1.2000$ USD/EUR
- Fair forward: $F^* = 1.2000 \times 0.9750/0.9600 = 1.21875$ USD/EUR

**Unhedged position:**
- Foreign PV: €$1{,}000{,}000 \times 0.9750 =$ €$975{,}000$
- Domestic PV: $\$1.20 \times$ €$975{,}000 = \$1{,}170{,}000$

**Hedge:** Short forward to sell €1,000,000 at $K = F^* = 1.21875$

**At maturity:**
- Bond pays €1,000,000
- Deliver into forward, receive $\$1{,}218{,}750$

**Hedged PV (discount at USD rate):**

$$PV_{\text{hedged}} = \$1{,}218{,}750 \times 0.9600 = \$1{,}170{,}000$$

**Result:** Hedged PV equals unhedged PV when struck at fair forward. The hedged foreign bond behaves like a domestic zero-coupon bond—you earn the domestic rate, not the foreign rate.

**Return interpretation:**

$$\text{Hedged return} = \frac{\$1{,}218{,}750}{\$1{,}170{,}000} = 1.0417 = \frac{1}{0.96}$$

The hedged investor earns the USD discount rate (4.17%), not the EUR rate. This is exactly what CIP predicts: the hedge transforms foreign-rate exposure into domestic-rate exposure.

### Example F: Rolling Forward Hedge (Carry Illustration)

**Setup:**
- Long €1,000,000 exposure over 3 months
- Hedge with 1-month forwards, rolled monthly
- Assume spot stays constant at 1.2000 USD/EUR
- Each 1-month forward: $F = 1.2020$ (forward points = +20 pips)

**Month 1:**
- Short forward at $K = 1.2020$
- At expiry, spot is 1.2000
- Short-forward payoff: $N_F \times (K - S) = 1{,}000{,}000 \times 0.0020 = \$2{,}000$

**Months 2 & 3:** Same result, $\$2{,}000$ each month.

**Total 3-month carry: $\$6{,}000$**

**Interpretation:** With spot unchanged, the hedger earns the forward points (reflecting the interest differential). This is the "carry" from the hedge—it's not free profit, but compensation for the rate differential between currencies.

---

## 29.10 Practical Notes and Common Pitfalls

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

---

## Summary

The FX forward market is not a forecasting market—it's an interest rate market in disguise. Every forward rate is pinned down by the spot rate and the interest differential across currencies. Understanding this is essential for anyone managing cross-currency exposures.

**Key takeaways:**

1. **CIP determines the forward:** $F = S \times P_F / P_D$ links FX forwards to discount curves.

2. **Forward points = interest differential:** The forward-to-spot ratio reflects relative funding costs, not currency forecasts.

3. **Foreign currency = asset with yield:** Treat foreign currency as earning the foreign risk-free rate; this makes CIP a special case of dividend-adjusted forward pricing.

4. **Hedging transforms rate exposure:** A hedged foreign bond earns the domestic rate, not the foreign rate—the hedge "converts" the rate exposure along with the currency exposure.

5. **PV formula:** $V = N_F(S P_F - K P_D)$, equivalently $V = N_F P_D(F^* - K)$.

6. **FX delta:** $\partial V / \partial S = N_F P_F$—the present value of the foreign leg.

7. **Quote direction matters:** Inverting the quote inverts the DF ratio. Mixing conventions creates spurious signals.

8. **Liquidity beyond 1 year:** Andersen and Piterbarg note FX forwards are "rarely liquid beyond maturities of one year"—longer hedges use cross-currency basis swaps (Chapter 30).

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
| 16 | What is the "carry" in an FX forward? | The interest differential embedded in $F/S$ |
| 17 | What does Andersen & Piterbarg say about FX forward liquidity? | Rarely liquid beyond ~1 year; longer horizons use xccy swaps |
| 18 | What is the relationship between 1-period xccy swap and FX forward? | They are identical contracts |
| 19 | What creates apparent arbitrage signals in CIP calculations? | Mixing compounding conventions (simple vs continuous) |
| 20 | How do bid-ask spreads affect CIP? | They widen the no-arbitrage band; small deviations are normal |

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

12. Design a verification test suite for an FX forward pricer (list inputs, outputs, checks).

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

---

## Source Map

### (A) Verified Facts (Source-Backed)

| Fact | Source |
|------|--------|
| CIP: $F_0 = S_0 e^{(r - r_f)T}$ | Hull, *Options, Futures, and Other Derivatives*, Ch 5, eq. 5.9 |
| "Interest rate parity relationship from international finance" | Hull Ch 5 |
| Foreign currency as asset with yield $r_f$ | Hull Ch 5 |
| "A foreign currency can be regarded as an investment asset paying a known yield" | Hull Ch 5 |
| Two-strategy replication argument for CIP | Hull Ch 5, Figure 5.1 |
| Numerical arbitrage example (USD/AUD) | Hull Ch 5, Example 5.6 |
| Corporate hedging example (ImportCo/ExportCo) | Hull Ch 1 |
| Forward value formula $f = S_0 e^{-qT} - K e^{-rT}$ | Hull Ch 5, eq. 5.7 |
| Forward delta for asset with yield $q$: $e^{-qT}$ | Hull Ch 19 |
| "For the delta of a forward foreign exchange contract, it is set equal to the foreign risk-free rate" | Hull Ch 19 |
| FX forwards link discount curves via arbitrage | Andersen & Piterbarg, *Interest Rate Modeling*, Vol 1 Ch 6 |
| "The market for foreign exchange (FX) forwards and cross-currency basis swaps imposes certain arbitrage constraints" | Andersen & Piterbarg Vol 1 |
| FX forward market "rarely liquid beyond maturities of one year" | Andersen & Piterbarg Vol 1 |
| One-period xccy basis swap = FX forward | Andersen & Piterbarg Vol 1 |
| Currency carry = overnight rate differential | Taleb, *Dynamic Hedging* |

### (B) Reasoned Inference (Derived from A)

| Inference | Derivation |
|-----------|------------|
| DF-form CIP: $F = S \cdot P_F / P_D$ | Direct translation from Hull's rate form using $P = e^{-rT}$ |
| PV formula: $V = N_F(S P_F - K P_D)$ | Discount each currency leg separately, apply Andersen's ZCB valuation |
| Quote inversion inverts DF ratio | Algebraic inversion of CIP formula |
| Forward points = $S(P_F/P_D - 1)$ | Subtract spot from CIP-derived forward |
| Hedged foreign bond earns domestic rate | CIP + hedge construction |

### (C) Flagged Uncertainties

| Item | Note |
|------|------|
| Forward points scaling (pips/points) | Not specified in sources; pair-dependent |
| FX spot settlement conventions (T+2, T+1) | Not specified in provided excerpts |
| NDF settlement mechanics | Not covered in sources |
| Triangular arbitrage details | Not explicitly sourced |
| Uncovered interest parity (UIP) relationship to CIP | Mentioned but not developed |

---

*Cross-references: Ch 2 (discount factors), Ch 3 (zero/forward rates), Ch 21 (cross-currency curves), Ch 30 (cross-currency swaps)*
