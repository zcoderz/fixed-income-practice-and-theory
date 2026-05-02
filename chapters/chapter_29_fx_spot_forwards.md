# Chapter 29: FX Spot and Forwards — Pricing via Interest Differentials

---

## Introduction

EUR/USD spot is 1.10. The 1-year forward is 1.12. Is the market predicting that the euro will appreciate against the dollar?

The intuitive answer—"yes, the forward is higher, so the market expects EUR to rise"—is wrong. The forward rate has almost nothing to do with market expectations of future spot rates. Instead, the forward is pinned down by a fundamental no-arbitrage relationship: **covered interest parity (CIP)**. The forward differs from spot because interest rates differ across currencies. If you could earn more by investing in dollars than in euros, that advantage must be exactly offset by the forward FX rate—otherwise, you could lock in a risk-free profit.

This insight has profound implications for anyone managing cross-currency exposures. When a U.S. investor hedges a European bond with FX forwards, they are not betting on currency direction—they are locking in the interest differential. When a corporation hedges a future foreign-currency payable, the “cost” of hedging is not a penalty; it is the interest differential (the opportunity cost of holding one currency versus another). Understanding FX forwards means understanding that currency markets and interest rate markets are linked by no-arbitrage.

A standard corporate hedging example is a known future GBP payment hedged with a GBP/USD forward. The forward removes uncertainty about the domestic settlement amount, but the forward price itself is governed by CIP.

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

Prerequisites: [Chapter 2](chapters/chapter_02_time_value_discount_factors_replication.md), [Chapter 3](chapters/chapter_03_zero_forward_par_rates_triangle.md), [Chapter 11](chapters/chapter_11_dv01_pv01_definitions_computation.md), [Chapter 21](chapters/chapter_21_cross_currency_curves.md).  
Follow-on: [Chapter 30](chapters/chapter_30_fx_swaps_cross_currency_swaps.md), [Chapter 31](chapters/chapter_31_multi_currency_risk.md), [Chapter 33](chapters/chapter_33_collateral_discounting_ois.md).

## Learning Objectives
- After this chapter, you can translate an FX spot/forward quote into the two-currency cashflows and a domestic PV.
- You can compute the fair forward $F^{\star}(t,T)$ from spot and discount factors, and interpret forward points.
- You can value an off-market FX forward and explain the sign and units of the PV.
- You can compute and interpret FX delta and curve DV01s with explicit bump objects, bump size (1 bp = $10^{-4}$), units, and sign.
- You can explain why an FX forward is not (in general) the market’s expected future spot, and what that means for hedging and carry.

---

## 29.1 Spot FX Quotes and Quote Direction

### 29.1.1 What the Spot Rate Represents

A spot FX rate $S_t^{D/F}$ is the price of one unit of foreign currency in units of domestic currency. If $D =$ USD and $F =$ EUR, then $S^{\text{USD/EUR}} = 1.10$ means one euro costs 1.10 dollars.

Quote direction is a convention, not a law of nature. To avoid ambiguity, this chapter fixes a single notation throughout:

- $S_t^{D/F}$ means **domestic currency units per 1 unit of foreign currency**.

So $S_t^{\text{USD/EUR}}=1.10$ means USD 1.10 per EUR 1.00.

> **Why this matters:** Risk systems must store both the currency pair and the quote direction. Mis-storing the inverse is a common source of sign errors in P&L, hedge ratios, and position reports. A system that treats EUR/USD as "EUR per USD" when the market quotes it as "USD per EUR" will produce inverted hedge ratios.

### 29.1.2 Inverting the Quote

The inverse quote simply flips the numerator and denominator:

$$\boxed{S_t^{F/D} = \frac{1}{S_t^{D/F}}}$$

Worked inversion in the chapter's notation: if $S_t^{\text{USD/EUR}} = 1.20$ (USD per EUR), then $S_t^{\text{EUR/USD}} = 1/1.20 = 0.8333$ (EUR per USD).

This seems trivial, but the inversion becomes critical when applying CIP formulas. As we will see, inverting the spot quote also inverts the ratio of discount factors in the forward pricing formula. Getting this wrong is one of the most common implementation errors in cross-currency systems.

> **Notation note: chapter symbol order vs market pair name.** Throughout this chapter, $S^{D/F}$ writes domestic *first*, foreign *second*, so $S^{\text{USD/EUR}}$ means "USD per EUR." The FX market, however, names pairs as **base/quote**, where the rate equals quote-currency per 1 unit of base currency. So the same number market participants call "EUR/USD = 1.20" appears in this chapter as $S^{\text{USD/EUR}}=1.20$. The two conventions agree on the *number*; they disagree on the *order of letters* in the pair label. When reconciling a system value with a market quote, always check whether the stored field is "$D/F$" (chapter) or "BASE/QUOTE" (market).

> **Desk Reality: FX Quote Conventions**
>
> Market pair names always follow **base/quote**: the rate equals quote-currency units per 1 unit of base currency. Which side of the pair is "base" is a market convention that varies by pair:
>
> | Pair (base/quote) | Base | Quote | Quote meaning |
> |-------------------|------|-------|---------------|
> | EUR/USD, GBP/USD, AUD/USD, NZD/USD | EUR, GBP, AUD, NZD | USD | USD per 1 unit of base |
> | USD/JPY, USD/CHF, USD/CAD | USD | JPY, CHF, CAD | Quote currency per 1 USD |
>
> When a trader says "EUR/USD is bid at 1.0950," they mean someone will buy euros at the rate of 1.0950 USD per EUR. When they say "Dollar-Yen is offered at 150.25," they mean someone will sell dollars at 150.25 JPY per USD.
>
> **The "Big Figure" shorthand:** Traders often drop the leading digits. "Cable is fifty-two, fifty-five" means GBP/USD is bid at 1.2752, offered at 1.2755. The "big figure" (1.27) is understood from context.

### 29.1.3 Cross Rates and Triangular Consistency

When trading crosses (non-USD pairs), the market derives rates from the USD legs. In the chapter's $S^{D/F}$ notation:

$$S^{\text{EUR/GBP}} = \frac{S^{\text{USD/GBP}}}{S^{\text{USD/EUR}}},$$

read as (EUR per GBP) = (USD per GBP) ÷ (USD per EUR), since (USD per GBP) × (EUR per USD) = (EUR per GBP).

Numerical example. Suppose 1 GBP costs USD 1.2750 and 1 EUR costs USD 1.0900 — that is, $S^{\text{USD/GBP}}=1.2750$ and $S^{\text{USD/EUR}}=1.0900$. Then

$$S^{\text{EUR/GBP}} = \frac{1.2750}{1.0900} = 1.1697 \text{ EUR per GBP.}$$

(Equivalently, 1 EUR ≈ 0.8550 GBP, which is what the market typically *quotes* under the BASE/QUOTE convention as "EUR/GBP ≈ 0.8550" — recall that the market pair name "EUR/GBP" reverses the slash relative to chapter notation; see the Notation Note in §29.1.2.)

Arbitrage keeps cross rates consistent. If the calculated cross differs from the quoted cross by more than transaction costs, traders profit by buying cheap and selling dear across the three pairs. This **triangular arbitrage** keeps the currency market internally consistent.

> **Practitioner Note:** Most retail and interbank platforms quote crosses directly, but the underlying liquidity is in the USD pairs. When you trade EUR/GBP, the dealer often executes EUR/USD and GBP/USD legs, earning the cross spread. This explains why cross pairs typically have wider bid-ask spreads than major USD pairs.

---

## 29.2 Settlement Mechanics and Value Dates

Understanding exactly *when* FX transactions settle is essential for pricing short-dated forwards and for operational accuracy. This section covers the practical mechanics that textbooks often omit.

### 29.2.1 Spot Value Date (Spot Lag)

The **spot value date** is when the two currency amounts actually exchange hands. The “spot lag” is often described as **T+N business days** from the trade date, but the value of $N$ is **pair-specific** and conventions can change over time—treat it as a parameter you must verify for the currency pair and trade date.

A practical algorithm:
1. Start from the trade date.
2. Add $N$ business days.
3. Ensure the resulting date is a business day in *both* settlement centers; if not, roll forward to the next joint business day.

In many major pairs, spot has historically been **two** business days after trade date (“T+2”), with notable exceptions—always check the market convention and calendars.

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
> Most FX desks use vendor calendar services (and internal static data) to determine valid settlement dates. Getting this wrong causes settlement fails.

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
| **Spot** | T+N (often T+2) | Standard |

The pricing from TOD to spot follows the same interest differential logic, but applied to very short periods. The difference between TOM and spot prices (the **Tom/Next swap**) is used to roll positions—covered briefly in Section 29.12 and fully in Chapter 30.

---

## 29.3 The FX Forward Contract

### 29.3.1 Contract Structure

A forward contract is an agreement made today to exchange assets at a future time $T$. An FX forward specifically exchanges two currencies:

- **Long FX forward (buy foreign):** At maturity $T$, receive $N_F$ units of foreign currency and pay $K \cdot N_F$ units of domestic currency, where $K$ is the delivery price (strike) agreed at inception.

A canonical use case is corporate hedging: a firm with a known future foreign-currency payable uses a forward to lock in the domestic amount it will pay (or receive) at the value date.

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

This is the standard forward payoff: you agreed to buy foreign currency at $K$, but the spot rate at maturity is $S_T$. If $S_T \gt K$, the forward is profitable (you bought cheap); if $S_T \lt K$, you overpaid relative to spot.

**Unit check:** Both $S_T$ and $K$ are in $D/F$ units, so $(S_T - K) \cdot N_F$ is in domestic currency. ✓

The opposite corporate problem is a known future foreign-currency receivable: the firm can **sell** the foreign amount forward (short the forward) to lock in its domestic proceeds.

---

## 29.4 Covered Interest Parity: The Core Pricing Relationship

### 29.4.1 The No-Arbitrage Argument

Under flat continuously-compounded risk-free rates in the two currencies, covered interest parity implies:

$$\boxed{F_0 = S_0 e^{(r - r_f)T}}$$

where $r$ is the domestic risk-free rate and $r_f$ is the foreign risk-free rate (both continuously compounded).

One intuition is to treat foreign currency as an asset that **earns the foreign risk-free rate**. FX forward pricing is then a special case of dividend-adjusted forward pricing, with the foreign rate playing the role of a continuous dividend yield.

### 29.4.2 The Replication Argument

Consider two strategies that each start with 1,000 units of foreign currency at time $0$ and end entirely in domestic currency at time $T$. If both strategies are riskless and self-financing, they must produce the same terminal amount of domestic currency—otherwise the market admits a riskless arbitrage.

**Strategy A (Invest foreign, then convert via forward):**
1. Today, invest 1,000 foreign at the foreign risk-free rate $r_f$. At $T$ this grows to $1{,}000\\,e^{r_f T}$ foreign.
2. Today, also enter a forward contract to sell exactly $1{,}000\\,e^{r_f T}$ foreign at rate $F_0$ at time $T$.
3. At $T$, deliver the foreign principal-plus-interest into the forward and receive $1{,}000\\,e^{r_f T}\\,F_0$ domestic.

**Strategy B (Convert at spot, then invest in domestic):**
1. Today, sell 1,000 foreign at spot rate $S_0$ for $1{,}000\\,S_0$ domestic.
2. Invest $1{,}000\\,S_0$ at the domestic risk-free rate $r$. At $T$ this grows to $1{,}000\\,S_0\\,e^{rT}$ domestic.

Both strategies are riskless (the forward in Strategy A locks in the future FX conversion), and both turn an initial 1,000 foreign into a known domestic amount at $T$. No-arbitrage therefore requires the two terminal amounts to match:

$$1{,}000 \cdot e^{r_f T} \cdot F_0 = 1{,}000 \cdot S_0 \cdot e^{rT}$$

Solving: $F_0 = S_0 e^{(r - r_f)T}$.

This is **covered** interest parity because the FX conversion at maturity is locked in via the forward—there is no currency risk in the replication.

### 29.4.3 Worked Arbitrage Example

Assume:
- foreign (AUD) rate = 3% continuous
- domestic (USD) rate = 1% continuous
- spot = 0.7500 USD per AUD
- maturity $T=2$ years

The CIP-implied forward is:

$$0.7500 \times e^{(0.01-0.03)\times 2} = 0.7206.$$

If the market forward were instead 0.7000 (too low), an arbitrage is:

1. Borrow 1,000 AUD at 3% for 2 years, convert to 750 USD and invest at 1%
2. Enter a forward to buy 1,061.84 AUD (the amount owed) for $1,061.84 \times 0.7000 = 743.29$ USD
3. The 750 USD grows to $765.15. Pay $743.29 to settle the forward, repay the AUD loan
4. **Risk-free profit: $765.15 - 743.29 = 21.87$ USD per 1,000 AUD**

The trade scales linearly with notional (subject to funding, credit, and execution constraints).

### 29.4.4 The Discount Factor Form

A more robust formulation uses discount factors, avoiding ambiguity about compounding conventions:

$$\boxed{F(t,T) = S_t \frac{P_F(t,T)}{P_D(t,T)}}$$

where:
- $P_D(t,T)$ is the domestic discount factor (price of a domestic zero-coupon bond paying 1 at $T$)
- $P_F(t,T)$ is the foreign discount factor

This is the basic no-arbitrage constraint that ties FX forwards to the two discount curves (and, at longer maturities, to the instruments used to build those curves).

**Equivalent notation (common in term-structure texts):** If $X(t)$ is the spot FX rate in domestic per foreign, and $P_d(t,T), P_f(t,T)$ are domestic and foreign discount factors, then the forward FX rate for delivery at $T$ is

$$X_T(t)=X(t)\frac{P_f(t,T)}{P_d(t,T)}.$$

With $X(t)=S_t$, $P_d=P_D$, and $P_f=P_F$, this is the same formula as above.

**Why prefer discount factors?** The discount factor form is convention-robust: it works regardless of whether rates are quoted as simple, semiannual, or continuous. The formula $F = S \cdot P_F / P_D$ simply says: the forward equals spot scaled by the ratio of "present value of 1 unit of foreign" to "present value of 1 unit of domestic."

**Unit check:** $P_F / P_D$ is dimensionless, so $F$ inherits the units of $S$ ($D/F$). ✓

**Reconciliation with rate form:** Under continuous compounding, $P_D = e^{-rT}$ and $P_F = e^{-r_f T}$, so:

$$\frac{P_F}{P_D} = \frac{e^{-r_f T}}{e^{-rT}} = e^{(r - r_f)T}$$

Multiplying by $S$ recovers the continuous-rate CIP expression.

---

## 29.5 Forward Points: What They Reveal

### 29.5.1 Definition and Mechanics

Forward points are the difference between the forward rate and the spot rate:

$$\text{Forward points} = F(t,T) - S_t$$

Using the CIP formula:

$$F(t,T) - S_t = S_t \left( \frac{P_F(t,T)}{P_D(t,T)} - 1 \right)$$

This shows that forward points are driven entirely by the ratio of discount factors—which reflects the interest rate differential between the two currencies.

**Check (small-tenor approximation):** For short tenors, you can linearize the discount-factor ratio. Under continuous compounding with year fraction $\tau$,

$$P_D(t,T)\approx e^{-r\tau},\quad P_F(t,T)\approx e^{-r_f\tau}\quad\Rightarrow\quad F-S \approx S\\,(r-r_f)\\,\tau.$$

Example: if $S=1.10$, $r-r_f=2\\%$, and $\tau=0.25$, then forward points are $\approx 1.10\times 0.02\times 0.25=0.0055$, i.e. about 55 pips when 1 pip $=0.0001$.

### 29.5.2 Premium vs Discount

- **Forward premium (in $D/F$ quotes):** If $F \gt S$, the foreign currency trades at a forward premium. This occurs when $P_F / P_D \gt 1$, meaning domestic rates are higher than foreign rates (equivalently, $P_D \lt P_F$ for the same maturity).

- **Forward discount (in $D/F$ quotes):** If $F \lt S$, the foreign currency trades at a forward discount. This occurs when foreign rates exceed domestic rates.

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

Forward points are often quoted in **pips** (“percentage in point”). Conventions vary by currency pair and venue, so treat “pip size” as a parameter:
- For pairs typically quoted to **4 decimals**, 1 pip is often **0.0001**.
- For JPY pairs typically quoted to **2 decimals**, 1 pip is often **0.01**.

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

Forward-point conventions differ across venues (sign, pip scaling, whether points are added/subtracted from spot). A critical point: **inverting the quote flips the sign interpretation.**

If $F^{D/F} \gt S^{D/F}$ (forward premium in domestic-per-foreign terms), then:

$$F^{F/D} = \frac{1}{F^{D/F}} \lt \frac{1}{S^{D/F}} = S^{F/D}$$

In the inverted quote, the forward is at a *discount*. Same economics, different sign—depending on which way you read the quote.

---

## 29.6 Why Forward ≠ Expected Future Spot

This distinction is fundamental and often misunderstood. The forward rate is an **arbitrage price** pinned down by spot and interest differentials. It can be written as an expectation under a *pricing measure*, but it should not be read as the market’s real-world forecast of where spot will be.

### 29.6.1 The Arbitrage Explanation

The forward rate is determined entirely by **current** spot and **current** interest rates:

$$F = S \cdot e^{(r - r_f)T}$$

No expectations about future spot rates enter this equation. The forward is the "fair" exchange rate that prevents risk-free arbitrage between borrowing, lending, and FX markets—*today*.

### 29.6.2 The Forward Rate Bias Puzzle

**Uncovered interest parity (UIP)** is the hypothesis that the expected change in spot offsets the interest differential, so an investor is indifferent on average between holding the high-rate currency (and being expected to lose on FX) and holding the low-rate currency (and being expected to gain on FX). One common test runs a regression of spot changes on the forward premium:

$$s_{t+1}-s_t = a + b\left(f_t-s_t\right)+\varepsilon_{t+1},$$

where $s_t=\log S_t^{D/F}$ and $f_t=\log F_t^{D/F}$. Under UIP, the slope $b$ should be close to $+1$: in our $D/F$ convention, $f_t - s_t \gt 0$ means the foreign currency trades at a forward premium (domestic rates higher than foreign), and UIP predicts a matching expected appreciation of the foreign currency, $\mathbb{E}[s_{t+1} - s_t] \approx f_t - s_t \gt 0$. Equivalently, the high-rate (domestic) currency is expected to depreciate by roughly the interest differential.

A common empirical finding is that the slope $b$ is well below $+1$, often close to zero or even negative. Translated into the $D/F$ convention: when the foreign rate is one percentage point higher than its usual level (so $f_t - s_t$ is *more negative* than usual, i.e., a wider forward discount on the foreign currency), the foreign currency tends on average to *appreciate further* rather than depreciate as UIP predicts. This is the **forward discount / forward premium puzzle** (sometimes called the Fama puzzle).

A practical translation is:
- **CIP** is a no-arbitrage constraint (within transaction costs and funding frictions).
- **UIP** is an empirical statement about *risk premia* and can fail.

Carry trades (borrow low-rate, invest high-rate) have historically earned positive average returns, but they are exposed to tail events (sharp reversals) that are not captured by the forward premium alone.

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

In discount factor notation:

**PV of receiving $N_F$ foreign at $T$:**

$$PV_{\text{receive}} = N_F \cdot S_t \cdot P_F(t,T)$$

**PV of paying $K \cdot N_F$ domestic at $T$:**

$$PV_{\text{pay}} = -K \cdot N_F \cdot P_D(t,T)$$

**Total domestic PV:**

$$\boxed{V_t^{(D)} = N_F \left( S_t \cdot P_F(t,T) - K \cdot P_D(t,T) \right)}$$

Interpretation: the domestic value of one foreign zero-coupon bond is $S_t \cdot P_F(t,T)$. Many texts write this as $\widetilde{P}_d(t,T)=X(t)P_f(t,T)$, where $X(t)$ is spot in domestic per foreign. An FX forward is therefore equivalent to a long foreign ZCB (converted to domestic at spot) plus a short domestic ZCB.

### 29.7.3 The Fair Forward and Zero-Value Condition

Define the *fair forward* (no-arbitrage forward rate):

$$F^{\star}(t,T) = S_t \frac{P_F(t,T)}{P_D(t,T)}$$

Substituting into the PV formula:

$$V_t^{(D)} = N_F \cdot P_D(t,T) \left( F^{\star}(t,T) - K \right)$$

This is the standard "discounted difference" form: **PV = notional × discount factor × (market forward − delivery price)**.

A forward struck at the fair forward has value (approximately) zero at inception; afterwards it is marked-to-market each day using current spot and curves.

**Sanity check:** If $K = F^{\star}$, then $V = 0$. The forward is struck at fair value at inception. ✓

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

**Hedge:** Enter a short forward (sell foreign forward) with $N_F = A_F$ and delivery price $K = F^{\star}(0,T)$.

At maturity:
- You receive $A_F$ foreign from your underlying exposure
- You deliver $A_F$ into the forward, receiving $K \cdot A_F$ domestic

Your domestic payoff is $K \cdot A_F = F^{\star}(0,T) \cdot A_F$, which is known today. The FX risk is eliminated.

### 29.8.2 The Interest Differential Interpretation

When you hedge at the fair forward, you lock in the spot rate *adjusted* by the interest differential:

$$K = F^{\star}(0,T) = S_0 \frac{P_F(0,T)}{P_D(0,T)}$$

If domestic rates exceed foreign rates, $P_D \lt P_F$, so $K \gt S_0$: you pay more per unit of foreign in the forward than at spot. This is the "cost" of hedging—but it's not a penalty. It exactly reflects that you could have earned higher domestic interest rather than foreign interest over the hedge period.

### 29.8.3 Hedging a Foreign Bond: The Return Transformation

A hedged foreign bond earns the **domestic** rate, not the foreign rate. This is not immediately obvious but follows directly from CIP.

**Example (simple-rate convention so that DF, rate, and realized return are all stated consistently):**
- Hold a EUR zero-coupon bond paying €1,000,000 at $T = 1$ year
- EUR discount factor: $P_F = 0.9750$, equivalent to $R_F = 1/P_F - 1 \approx 2.56\\%$ over the year
- USD discount factor: $P_D = 0.9600$, equivalent to $R_D = 1/P_D - 1 \approx 4.17\\%$ over the year
- Spot: $S_0 = 1.2000$ USD/EUR

**Unhedged:** Your USD value at maturity depends on the unknown future spot rate.

**Hedged:** Sell €1,000,000 forward at $F^{\star} = S_0 \cdot P_F/P_D = 1.2000 \times 0.9750/0.9600 = 1.21875$ USD/EUR.

At maturity, receive $1{,}218{,}750$ USD. The USD value invested today (the cost of the EUR ZCB in USD) is $S_0 \cdot P_F \cdot \text{€1{,}000{,}000} = 1.2000 \times 0.9750 \times 1{,}000{,}000 = 1{,}170{,}000$ USD. Your hedged 1-year USD return is:

$$\frac{1{,}218{,}750}{1{,}170{,}000} = 1.0417,$$

i.e., $4.17\\%$ over the year. This matches $R_D = 1/P_D - 1 = 4.17\\%$, the **USD** simple rate—not the EUR rate. (Equivalently in continuous compounding, $\ln(1.0417)\approx -\ln(P_D)\approx 4.08\\%$—same fact, different convention.)

**The hedge transforms your rate exposure along with your currency exposure.** You "converted" the EUR bond into a synthetic USD bond.

### 29.8.4 A Key Insight on Hedging

Hedging is about reducing uncertainty, not maximizing realized return. There is no guarantee that the hedged outcome will be better than the unhedged outcome in hindsight.

The value of hedging is certainty, not expected return. The hedger gives up the chance of favorable spot movements in exchange for eliminating the risk of unfavorable ones.

---

## 29.9 Risk Sensitivities of an FX Forward

### 29.9.1 FX Delta (Spot Sensitivity)

Differentiating the PV formula with respect to spot $S_t$ (holding curves fixed):

$$\boxed{\frac{\partial V_t^{(D)}}{\partial S_t} = N_F \cdot P_F(t,T)}$$

Under the standard “foreign currency earns the foreign risk-free rate” interpretation, the forward’s spot delta equals the **present value of the foreign notional**. Under flat continuously-compounded rates, $P_F(t,T)=e^{-r_f (T-t)}$, so $\partial V/\partial S = N_F e^{-r_f (T-t)}$.

**Units:** $V^{(D)}$ is in domestic currency and $S$ is in domestic-per-foreign, so $\partial V/\partial S$ has units of **foreign currency** (e.g., EUR). Some risk systems report a “domestic delta” by multiplying by spot; always check your report’s units.

**Check (spot hedge sizing):** If you are long the forward (buy foreign), your spot delta is $+N_F P_F(t,T)$ in foreign units. A first-order spot hedge is therefore to **sell** $N_F P_F(t,T)$ units of foreign in spot (so the combined $dV\approx (N_F P_F - N_{\text{spot}})\\,dS$ is near zero). The remaining risk is then mainly curve/basis risk and any mismatch in bump/mark conventions.

> **Desk Reality:** Many risk reports show FX delta in *foreign units* as $N_F P_F(t,T)$, not the contractual notional $N_F$.
> **Common break:** Spot hedges are sized off notional and ignore $P_F$, creating a tenor-dependent residual delta.
> **What to check:** Confirm whether your system reports delta in foreign units, domestic units, or “per 1% move,” and reconcile via unit conversions.

### 29.9.2 Interest Rate Sensitivities

From the PV formula $V = N_F(S_t P_F - K P_D)$, you can read the forward as a **two-bond position**:
- long a foreign ZCB worth $N_F \cdot S_t \cdot P_F(t,T)$ in domestic currency,
- short a domestic ZCB worth $N_F \cdot K \cdot P_D(t,T)$.

**Risk bump definitions (this chapter):**
- **Bump objects:** the discount curves used to compute $P_D(t,T)$ and $P_F(t,T)$. (In real systems: bump curve pillars and rebuild; same idea.)
- **Bump size:** 1 bp = $10^{-4}$.
- **Units:** domestic currency per 1 bp.
- **Sign convention (book-wide):** $DV01 := PV(\text{rates down }1\text{bp}) - PV(\text{base})$ for the stated bump object.
Let $\tau(t,T)$ be the year fraction implied by the curve’s day count between dates $t$ and $T$.

**Foreign curve DV01 (toy single-maturity approximation):** if you bump the foreign continuously-compounded zero rate down by 1bp so that $P_F$ increases by $\Delta P_F \approx \tau(t,T)P_F \cdot 10^{-4}$, then

$$DV01_F \approx +N_F \cdot S_t \cdot \tau(t,T) \cdot P_F(t,T)\cdot 10^{-4}.$$

**Domestic curve DV01 (toy single-maturity approximation):** if you bump the domestic continuously-compounded zero rate down by 1bp so that $P_D$ increases by $\Delta P_D \approx \tau(t,T)P_D \cdot 10^{-4}$, then

$$DV01_D \approx -N_F \cdot K \cdot \tau(t,T) \cdot P_D(t,T)\cdot 10^{-4}.$$

These two curve risks can partially offset, but the sign and magnitude depend on $S_t$, $K$, and the relative curve levels.

**Check (at-market forward, DV01 magnitudes match):** For a forward struck at the fair rate $K=F^{\star}(t,T)$, no-arbitrage implies $K\\,P_D(t,T)=S_t\\,P_F(t,T)$. Plugging this into the toy single-maturity DV01 expressions shows $|DV01_F|\approx |DV01_D|$ (opposite signs). In that toy setting, a simultaneous 1bp down shift to **both** curves has a much smaller net PV impact than bumping one curve alone—the forward is primarily exposed to **relative** moves between the curves (a cross-currency “basis” move), not just level.

### 29.9.3 Delta of Forward vs Delta of Spot Position

Forwards have deltas even though they are not options. For a forward on a non-dividend-paying equity, the delta is 1: one unit of forward behaves like one unit of spot plus a financing adjustment. For FX forwards, the analogous result is $\Delta_{FX}=e^{-r_f (T-t)}$ because holding foreign currency earns the foreign risk-free rate (the “yield” on the underlying).

---

## 29.10 Non-Deliverable Forwards (NDFs)

### 29.10.1 What Is an NDF?

A **non-deliverable forward (NDF)** is an FX forward where, at maturity, there is no physical exchange of the two currencies. Instead, the contract is **cash-settled** in a specified settlement currency (USD in our examples) using a published reference **fixing**.

**Why NDFs exist:** Some currencies are restricted or operationally difficult to deliver offshore. An NDF lets two parties hedge or take exposure without moving the restricted currency through the offshore settlement system.

### 29.10.2 Cash-Settlement Mechanics (Be Explicit About Quote Direction)

Economically, an NDF settles the payoff of the corresponding deliverable forward. The only “gotcha” is that the cash amount depends on **(i) quote direction** and **(ii) which currency the notional is specified in**. Always reconcile this with the trade confirmation.

Two common parameterizations:

**(1) Chapter convention (domestic per foreign, notional in foreign):**
- Quotes $S^{D/F}$ and $K^{D/F}$ are **domestic currency per 1 foreign**.
- Notional is $N_F$ in the foreign currency.
- A long forward to *buy foreign* (receive $N_F$, pay $K N_F$ domestic) has domestic cash payoff at fixing:

$$\boxed{\text{Payoff in }D = N_F\left(S_{\text{fix}}^{D/F} - K^{D/F}\right)}$$

and the short position flips the sign.

**(2) Common NDF quote direction (foreign per USD, notional in foreign):**
- Quotes $S_{\text{fix}}$ and $K$ are **foreign currency per 1 USD** (e.g., KRW per USD).
- Notional is $N_F$ in the foreign (restricted) currency.
- If you are *selling foreign / buying USD forward*, the USD cash payoff is:

$$\boxed{\text{Payoff in USD} = N_F\left(\frac{1}{K}-\frac{1}{S_{\text{fix}}}\right)}$$

(again, the opposite position flips the sign).

Both formulas are the same economics written under different quote directions. A good unit check is that the payoff currency is the settlement currency and the payoff has “notional × price difference” structure.

**Check (reconciling the two payoffs):** If the market quote is “foreign per USD,” call it $Q := S^{F/D}$, then the inverse quote is $S^{D/F}=1/Q$ (and similarly $K^{D/F}=1/K^{F/D}$). Substituting $S_{\text{fix}}^{D/F}=1/S_{\text{fix}}$ and $K^{D/F}=1/K$ into the chapter-convention payoff $N_F(S_{\text{fix}}^{D/F}-K^{D/F})$ gives $N_F(1/S_{\text{fix}}-1/K)$, which matches the “foreign per USD” expression up to sign. That sign difference is exactly the trade-direction difference: “buy foreign” vs “buy USD.” This is why confirmations must state quote direction, notional currency, and settlement currency explicitly.

**PV note:** Because an NDF settles as a single cash amount in the settlement currency, the mark-to-market at time $t$ discounts that settlement-currency cash amount using the settlement-currency discount curve (the same “single-currency PV” idea as any cash-settled forward).

### 29.10.3 Worked NDF Example (Foreign per USD Quote)

**Contract:** sell KRW $N_F=1{,}000{,}000{,}000$ vs USD, cash-settled in USD.  
**Quote direction:** KRW per USD.  
**Contract rate:** $K=1320$ KRW/USD.  
**Fixing:** $S_{\text{fix}}=1350$ KRW/USD (KRW weakened).

Using the “foreign per USD” formula above,

$$\text{Payoff in USD}=N_F\left(\frac{1}{K}-\frac{1}{S_{\text{fix}}}\right)=1{,}000{,}000{,}000\left(\frac{1}{1320}-\frac{1}{1350}\right)\approx +16{,}835.$$

Interpretation: you locked in **more USD per KRW** than the fixing implies, so you receive a positive USD settlement.

### 29.10.4 Fixing Risk

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

### 29.11.1 The Components of FX Forward P&L

Total P&L from an FX forward position decomposes into:

1. **Spot P&L:** Change in forward value due to spot rate movement
2. **Forward Point (Curve) P&L:** Change due to interest rate differential movement
3. **Time/Carry P&L:** Change from passage of time (forward converging to spot)
4. **Curve Shape P&L:** Changes in yield curve shape (beyond parallel shifts)

### 29.11.2 First-Order Decomposition

For small changes, the PV change decomposes as:

$$\Delta V \approx \underbrace{N_F \cdot P_F \cdot \Delta S}_{\text{Spot PnL}} + \underbrace{N_F \cdot S \cdot \Delta P_F - N_F \cdot K \cdot \Delta P_D}_{\text{Rates PnL}}$$

**Spot P&L** is intuitive: the position gains/loses based on spot movement times delta.

**Rates P&L** captures the impact of domestic and foreign rate changes on the discount factors.

### 29.11.3 Carry: The Time-Decay Component

Unlike options, forwards don't have time decay in the traditional sense. But they do have **carry**—the P&L from time passing if nothing else changes.

If you're long a forward at $K \lt F^{\star}_t$, as time passes the discount factor approaches 1 so the present value $N_F\\,P_D(t,T)\\,(F^{\star}_t-K)$ pulls toward its undiscounted value $N_F(F^{\star}_t-K)$; in the limit $t\to T$, $F^{\star}_t \to S_t$, so the realized payoff at maturity is $N_F(S_T-K)$. The rate at which $V_t$ changes from time alone (with spot and curves held fixed) is the **carry**.

A direct calculation from $V_t = N_F (S_t P_F(t,T) - K P_D(t,T))$, holding $S_t$ and the curves fixed and using $\partial P_F/\partial t = r_f P_F$, $\partial P_D/\partial t = r P_D$ under flat continuous rates, gives

$$\frac{\partial V_t}{\partial t}\bigg|_{K=F^{\star}_0} = N_F\\,K\\,P_D(t,T)\\,(r_f - r),$$

i.e., the **long-forward carry has the sign of $r_f - r$**: positive when foreign rates exceed domestic rates, negative when domestic rates are higher. For short tenors $K\\,P_D \approx S$, so a useful daily approximation is

$$\boxed{\text{Daily carry of a long FX forward} \approx N_F \times S \times (r_f - r) \times \frac{1}{365}.}$$

A short forward simply flips the sign. This is the FX-market analogue of "earn the foreign rate, pay the domestic rate" on a synthetic-long-foreign position—when domestic rates are higher than foreign, that synthetic position bleeds carry every day.

### 29.11.4 Worked P&L Attribution Example

**Position:** Long EUR 10mm vs USD, struck at the Day 0 fair forward.

**Day 0 (consistent inputs):**
- Spot $S_0 = 1.0800$
- USD 3M continuously-compounded rate: $r = 5.5\\%$
- EUR 3M continuously-compounded rate: $r_f = 4.0\\%$
- Tenor $T = 0.25$ years
- Fair forward $F^{\star}_0 = S_0\\,e^{(r-r_f)T} = 1.0800 \times e^{0.00375} \approx 1.0841$
- Strike $K = F^{\star}_0 = 1.0841$, so $V_0 = 0$

**Day 30:**
- Spot $S_{30} = 1.0900$ (+100 pips)
- USD rate: $5.6\\%$ (+10 bp)
- EUR rate: $3.9\\%$ (-10 bp)
- Remaining tenor: $T - t = 60/365 \approx 0.1644$ years
- New fair forward $F^{\star}_{30} = 1.0900 \times e^{0.017 \times 0.1644} \approx 1.0930$
- Discount factors: $P_D^{30} \approx e^{-0.056 \times 0.1644} \approx 0.9908$, $P_F^{30} \approx e^{-0.039 \times 0.1644} \approx 0.9936$

Direct revaluation:

$$V_{30} = N_F\\,(S_{30}\\,P_F^{30} - K\\,P_D^{30}) = 10^7\\,(1.0900\times0.9936 - 1.0841\times0.9908) \approx +88{,}900 \text{ USD}.$$

**P&L attribution (first-order, evaluated at Day-0 levels):**

1. **Spot P&L:** $N_F\\,P_F\\,\Delta S \approx 10^7 \times 0.9901 \times 0.01 \approx +99{,}000$ USD. (Day-0 $P_F = e^{-0.04\times 0.25} \approx 0.9901$.)

2. **Rates P&L:** USD rates up 10 bp and EUR rates down 10 bp widen $r-r_f$ by 20 bp, lifting the fair forward and helping a long position struck at the (now lower) Day 0 fair. Using the toy DV01s of Section 29.9.2 evaluated at $T-t = 0.25$:

$$\Delta V_{r_f} \approx N_F\\,S_0\\,(T-t)\\,P_F^0 \times 10 \text{ bp} \approx +2{,}700 \text{ USD};$$

$$\Delta V_{r_d} \approx N_F\\,K\\,(T-t)\\,P_D^0 \times 10 \text{ bp} \approx +2{,}700 \text{ USD};$$

total Rates P&L $\approx +5{,}400$ USD.

3. **Carry P&L (negative for a long forward when domestic rates exceed foreign):** $K = F^{\star}_0 \gt S_0$, and with rates unchanged the fair forward decays toward spot, so a long position at fair *loses* about $N_F\\,S\\,(r_f - r)/365$ per day. Over 30 days at $r_f - r = -1.5\\%$:

$$\text{Carry} \approx 10^7 \times 1.0800 \times (-0.015) \times \tfrac{30}{365} \approx -13{,}300 \text{ USD}.$$

4. **Total (first-order):** $99{,}000 + 5{,}400 - 13{,}300 \approx +91{,}100$ USD.

This matches the direct revaluation of $\approx +88{,}900$ within $\sim$2.5%; the residual is the second-order cross-term $N_F\\,(\Delta S)(\Delta P_F)$ and discounting effects, which a fully consistent attribution allocates to a small "cross-effects" bucket. (Exact attribution requires full revaluation at intermediate points, or an Itô-style decomposition that includes second-order terms.)

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

### 29.12.3 Implied Overnight Rate Differential

T/N points reflect one day of interest differential between the two currencies. From them, you can back out the implied **overnight rate differential** (not a single overnight rate):

$$\text{Implied annualized }(r_d - r_f) \approx \frac{\text{T/N points}}{S} \times 360,$$

where the right-hand side is a decimal (multiply by 100 if you prefer percent). The "360" is the day-count basis used in the major money markets that quote T/N points; for currencies on an ACT/365 basis (e.g., GBP, AUD), use 365 instead. Given a known overnight rate in one currency (e.g., SOFR for USD), the implied rate in the other currency follows by addition or subtraction.

This should approximately equal the actual overnight interest differential between the two OIS curves. Deviations indicate funding stress or year-end effects (and feed directly into the cross-currency basis covered in Chapter 30).

### 29.12.4 Connection to Cross-Currency Basis

Persistent deviations between T/N implied rates and actual OIS rates reflect **cross-currency basis**—the topic of Chapter 30. When dollar funding is scarce, T/N points widen beyond what pure interest differentials would suggest.

**Full treatment of FX swaps, cross-currency swaps, and the cross-currency basis appears in Chapter 30.**

---

## 29.13 Worked Examples

> **Note:** Numbers are illustrative. Settlement conventions (T+2, T+1, etc.) and day-count conventions must be adapted to specific currency pairs in practice.

### Example A: Computing the Forward from Rates

**Given:**
- Spot: $S_0^{\text{USD/EUR}} = 1.2000$ (1.20 USD per EUR)
- USD rate: $r = 5\\%$ continuous
- EUR rate: $r_f = 2\\%$ continuous
- Maturity: $T = 0.5$ years

**Compute forward:**

$$F_0 = S_0 e^{(r - r_f)T} = 1.2000 \times e^{(0.05 - 0.02) \times 0.5} = 1.2000 \times e^{0.015}$$

$$e^{0.015} \approx 1.0151$$

$$F_0 \approx 1.2181 \text{ USD/EUR}$$

**Forward points:** $F_0 - S_0 = 1.2181 - 1.2000 = 0.0181$ USD/EUR = 181 pips

**Interpretation:** USD rates exceed EUR rates, so EUR trades at a forward premium (more dollars per euro in the forward than at spot).

### Example B: 3M EUR/USD Forward — Fair Forward, PV, and Risk

**Context.** A USD-based investor will pay EUR 10,000,000 in 3 months and wants to lock the USD amount.

**Timeline (illustrative; ignore holidays).**
- Trade date: 2026-02-17
- Spot value date: 2026-02-19 (assume T+2)
- Forward value date: 2026-05-19 (3M from spot)
- Exchange at maturity: physical exchange of EUR and USD on 2026-05-19

**Inputs.**
- Spot (for the spot value date): $S_0^{\text{USD/EUR}} = 1.2000$.
- Discount factors from spot date to forward date (toy numbers):
  - USD: $P_D=0.9901$
  - EUR: $P_F=0.9975$
  These are consistent with simple money-market rates $R_D=4\\%$, $R_F=1\\%$ over $\tau=0.25$ via $P=1/(1+R\tau)$.
- Notional: $N_F = 10{,}000{,}000$ EUR.
- Contract delivery price: $K=1.2100$ USD/EUR (slightly above fair).

**Outputs.** Fair forward $F^{\star}$; PV in USD at the trade date (using the spot-to-forward discount factors); FX delta in foreign units; curve DV01s in USD per 1 bp under the bump object and sign of Section 29.9.

**Step 1 — fair forward (CIP, DF form).**

$$F^{\star} = S_0 \frac{P_F}{P_D} = 1.2000 \times \frac{0.9975}{0.9901} = 1.2090.$$

**Step 2 — PV of the off-market forward, long (buy EUR).**

$$V_0^{(\text{USD})}=N_F\left(S_0P_F-KP_D\right)=10{,}000{,}000\left(1.2000\times0.9975-1.2100\times0.9901\right)\approx-10{,}200.$$

The PV is *negative* because $K=1.2100$ is above the fair $F^{\star}=1.2090$: the long forward overpays for EUR by about $0.001$ USD per EUR, applied to the discounted notional.

**Step 3 — FX delta (foreign units).**

$$\frac{\partial V}{\partial S}=N_F P_F=10{,}000{,}000\times0.9975=9{,}975{,}000\\,\text{EUR}.$$

**Step 4 — curve DV01s** (toy parallel bump on each curve separately; $1\\,\text{bp}=10^{-4}$; $\tau=0.25$).

$$DV01_F \approx +N_F\\,S_0\\,\tau\\,P_F\\,10^{-4}=+10{,}000{,}000\times1.2000\times0.25\times0.9975\times10^{-4}\approx +299.$$

$$DV01_D \approx -N_F\\,K\\,\tau\\,P_D\\,10^{-4}=-10{,}000{,}000\times1.2100\times0.25\times0.9901\times10^{-4}\approx -300.$$

**Cashflows.**
| Date | Cashflow | Explanation |
|---|---:|---|
| 2026-05-19 | +€10,000,000 | Receive foreign notional |
| 2026-05-19 | -USD 12,100,000 | Pay domestic notional $K\times N_F$ |

**P&L and risk interpretation.**
- **Spot move:** if EUR/USD rises by $+0.01$, PV increases by about $\Delta V \approx (N_F P_F)\Delta S \approx 9.975\times 10^6 \times 0.01 \approx +99{,}750$ USD.
- **Rates move:** a 1 bp down shift in EUR discounting increases PV by $\approx +299$ USD; a 1 bp down shift in USD discounting changes PV by $\approx -300$ USD (i.e., decreases PV by about USD 300).
- **Desk mapping:** it is natural to treat the forward as "long foreign ZCB, short domestic ZCB" plus spot delta; many risk systems bucket the two DV01s onto the respective curves.

**Sanity checks.**
- Units: $S, K$ are USD/EUR, so $N_F(SP_F-KP_D)$ is in USD.
- Zero-value check: if $K=F^{\star}$, PV is (approximately) 0 at inception.
- Quote-direction check: if you invert the quote, invert the DF ratio as well.

### Example C: Arbitrage When Forward Is Mispriced

**Given:** Same as the worked example above, but market forward is $F_{\text{mkt}} = 1.2200$ (too high vs CIP-implied 1.2090).

**Arbitrage strategy:**

1. **Borrow USD:** $1.2000$ at 4% for 0.25 years → repay $1.2000 \times 1.01 = 1.212$ USD

2. **Buy EUR at spot:** $1.20$ USD buys €1.00

3. **Invest EUR:** €1.00 at 1% for 0.25 years → receive €$1.0025$

4. **Sell EUR forward:** Deliver €1.0025 at $F_{\text{mkt}} = 1.2200$ → receive $1.2231$ USD

**Net at maturity:**
- Receive: $1.2231$ USD
- Repay: $1.2120$ USD
- **Profit: $0.0111$ USD per €1 initial**

Scaled to €10,000,000: profit = $111,000$ USD.

This is risk-free profit, so arbitrageurs would trade until $F_{\text{mkt}} = F^{\star}$.

---

## 29.14 Practical Notes and Common Pitfalls

### Quote Direction Errors

The most common implementation error is inverting the quote without inverting the DF ratio:

$$F^{F/D} = S^{F/D} \times \frac{P_D}{P_F}$$

Note the ratio flips when the quote direction flips. Systems that store currency pairs inconsistently will produce wrong hedge ratios.

> **Pitfall — Quote inversion in CIP:** Inverting $S^{D/F}$ to $S^{F/D}$ but forgetting to invert the discount-factor ratio in $F = S\\,P_F/P_D$.
> **Why it matters:** You can flip the sign of forward points and the direction of hedges/P&L.
> **Quick check:** Compute $F$ in the stored quote direction and verify units; then invert both sides and check the inverted-quote formula uses $P_D/P_F$.

### Compounding Convention Mismatch

If you use continuous-rate CIP ($F = S e^{(r - r_f)T}$) but plug in simple rates, you get spurious "arbitrage" signals.

**Example:** With $T = 2$ years, simple rates $R_D = 10\\%$, $R_F = 2\\%$:
- Correct (simple): $F = 1.2 \times (1.20/1.04) = 1.385$
- Wrong (treating as continuous): $F = 1.2 \times e^{0.16} = 1.408$

The 2.3-cent difference is not arbitrage—it's convention mismatch. **Always use discount factors to avoid this trap.**

### Settlement Timing

The forward settlement date determines $T$. If the market defines "spot" as settling T+2, ensure the forward's time-to-maturity is measured consistently.

### Bid-Ask Spreads

Forward points have bid-ask spreads; so does spot. The no-arbitrage band has width proportional to transaction costs. CIP holds within the band; observed deviations are often within bid-ask.

### FX Forward Liquidity

Liquidity in outright FX forwards generally falls off with maturity. Longer-dated hedges are often executed via FX swaps or cross-currency swaps (Chapter 30) rather than single-period outrights.

### Verification Checklist

| Test | What to Check |
|------|---------------|
| **Dimension/units** | $S$, $F$, $K$ all in same units ($D/F$). $P_D$, $P_F$ dimensionless. |
| **CIP replication** | Reproduce $F$ from borrow/lend/spot; confirm no free profit. |
| **Zero-value test** | $V(K = F^{\star}) = 0$ when curves are consistent. |
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

6. **PV formula:** $V = N_F(S P_F - K P_D)$, equivalently $V = N_F P_D(F^{\star} - K)$.

7. **FX delta:** $\partial V / \partial S = N_F P_F$—the present value of the foreign leg.

8. **Quote direction matters:** Inverting the quote inverts the DF ratio. Mixing conventions creates spurious signals.

9. **NDFs settle in cash:** For restricted currencies, no physical exchange occurs; settlement is based on a fixing rate.

10. **Liquidity at longer maturities:** Outright forwards are typically most liquid at short tenors; longer hedges often use FX swaps or cross-currency swaps (Chapter 30).

---

## Key Concepts

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| Spot FX rate $S^{D/F}$ | Domestic currency per 1 foreign | Foundation for all FX calculations; quote direction critical |
| CIP (continuous form) | $F = S e^{(r - r_f)T}$ | No-arbitrage link between spot, forward, and rates |
| CIP (DF form) | $F = S \cdot P_F / P_D$ | Convention-robust; avoids compounding ambiguity |
| Forward points | $F - S$ | Reflects interest differential, not forecast |
| Forward premium/discount | $F \gt S$ or $F \lt S$ | Higher domestic rates → forward premium |
| FX forward PV | $N_F(S P_F - K P_D)$ | Discount each currency leg, convert to domestic |
| Fair forward | $F^{\star} = S \cdot P_F / P_D$ | Delivery price that makes PV = 0 at inception |
| FX delta | $N_F \cdot P_F$ | Spot sensitivity of forward position |
| NDF | Cash-settled FX forward | Used for restricted currencies |
| Fixing risk | Risk that fixing differs from market | Unique to NDFs; managed via hedging/diversification |
| Value date | Settlement date for an FX exchange | Spot lag is pair-specific (often described as T+N) and must be a joint business day in both centers |

---

## Notation

| Symbol | Definition |
|--------|------------|
| $D$, $F$ | Domestic and foreign currencies |
| $S_t^{D/F}$ | Spot FX at time $t$ (domestic per 1 foreign) |
| $F(t,T)$ | Forward FX rate for maturity $T$ |
| $K$ | Delivery price (strike) of FX forward |
| $P_D(t,T)$ | Domestic discount factor |
| $P_F(t,T)$ | Foreign discount factor |
| $X(t)$ | Alternative notation for spot FX in domestic per foreign (some texts) |
| $P_d(t,T)$, $P_f(t,T)$ | Alternative notation for domestic/foreign discount factors (aliases for $P_D$, $P_F$) |
| $X_T(t)$ | Alternative notation for the forward FX rate for delivery at $T$ (alias for $F(t,T)$ here) |
| $\widetilde{P}_d(t,T)$ | Domestic value of a foreign ZCB paying 1 foreign at $T$ |
| $\tau(t,T)$ | Year fraction between dates $t$ and $T$ (curve day count) |
| $N_F$ | Foreign currency notional |
| $r$, $r_f$ | Domestic and foreign risk-free rates |
| $DV01_D$, $DV01_F$ | FX-forward DV01s for domestic/foreign discount curves (as defined in Section 29.9) |
| $S_{\text{fix}}$ | NDF fixing rate |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What does $S_t^{D/F}$ represent? | Domestic currency units per 1 foreign unit |
| 2 | State CIP in discount factor form. | $F = S \cdot P_F / P_D$ |
| 3 | State CIP in continuous rate form. | $F = S \cdot e^{(r - r_f)T}$ |
| 4 | Why can foreign currency be treated as an asset with yield? | Holding foreign currency earns the foreign risk-free rate |
| 5 | If domestic rates > foreign rates, is the forward at premium or discount? | Premium ($F \gt S$ in $D/F$ quotes) |
| 6 | What are "forward points"? | $F - S$; the forward-to-spot difference |
| 7 | What drives forward points? | The interest differential between currencies |
| 8 | What is the PV formula for a long FX forward? | $V = N_F(S \cdot P_F - K \cdot P_D)$ |
| 9 | When is an FX forward worth zero at inception? | When $K = F^{\star}$ (struck at the fair forward) |
| 10 | What is the FX delta of a forward? | $N_F \cdot P_F$ |
| 11 | How do you hedge a known future foreign cash inflow? | Short forward (sell foreign forward) with $N_F =$ inflow amount |
| 12 | What return does a hedged foreign ZCB earn? | Approximately the domestic rate, not the foreign rate |
| 13 | What happens to CIP formulas when you invert the quote? | The DF ratio inverts: $F^{F/D} = S^{F/D} \cdot P_D / P_F$ |
| 14 | Why is the DF form of CIP preferred? | It's convention-robust; avoids compounding confusion |
| 15 | What is "covered" about covered interest parity? | The FX conversion at maturity is locked in via a forward |
| 16 | Does the forward rate predict future spot rates? | Not in general; it is an arbitrage price from spot and rates (a pricing-measure expectation), not a physical forecast |
| 17 | What is an NDF? | A cash-settled FX forward for restricted currencies |
| 18 | What is fixing risk in NDFs? | Risk that the official fixing rate diverges from market rates |
| 19 | How do you determine the spot value date? | Apply the pair-specific spot lag (often described as T+N) and roll to a joint business day in both centers |
| 20 | What is a pip in FX? | 0.0001 for most pairs; 0.01 for JPY pairs |
| 21 | What creates apparent arbitrage signals in CIP calculations? | Mixing compounding conventions (simple vs continuous) |
| 22 | How do bid-ask spreads affect CIP? | They widen the no-arbitrage band; small deviations are normal |
| 23 | What is the "forward rate bias" puzzle? | High-rate currencies tend to appreciate, contrary to forward prediction |
| 24 | What is a Tom/Next swap used for? | Rolling spot positions to avoid physical delivery |
| 25 | Why might longer-dated hedges use FX swaps or cross-currency swaps rather than outrights? | Outright forward liquidity often declines with tenor; swaps can be the more practical way to express longer-horizon funding/hedging |

---

## Mini Problem Set

### Problems

1. Given $S^{\text{USD/EUR}} = 1.2500$, compute $S^{\text{EUR/USD}}$.

2. Show that inverting the quote inverts the DF ratio: if $F^{D/F} = S^{D/F} \cdot P_F/P_D$, derive $F^{F/D}$.

3. With $S_0 = 1.10$ USD/EUR, $P_D(0, 0.5) = 0.98$, $P_F(0, 0.5) = 0.995$, compute $F(0, 0.5)$.

4. Compute PV (in USD) of a long forward with $N_F = 5{,}000{,}000$ EUR, $S_0 = 1.10$, $K = 1.12$, $P_D = 0.99$, $P_F = 0.995$.

5. Explain (in words) the arbitrage strategy when $F_{\text{mkt}} \gt F^{\star}$.

6. Using simple rates $R_D$, $R_F$, derive $F = S(1 + R_D T)/(1 + R_F T)$ from first principles.

7. If USD rates rise (all else equal), what happens to the USD/EUR forward?

8. A U.S. investor will receive €10 million in 6 months. What forward position hedges the FX risk?

9. Re-do Problem 4 using the formula $V = N_F \cdot P_D \cdot (F^{\star} - K)$; verify it matches.

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
If $F_{\text{mkt}} \gt F^{\star}$: borrow domestic, convert to foreign at spot, invest foreign, sell foreign forward at the high rate. At maturity, the forward proceeds exceed the domestic borrowing cost, locking in risk-free profit.

**Problem 9:**
$F^{\star} = 1.10 \times 0.995/0.99 = 1.1056$
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

- Andersen & Piterbarg, *Interest Rate Modeling* (FX forwards, discount-factor parity, cross-currency arbitrage constraints)
- Musiela & Rutkowski, *Martingale Methods in Financial Modelling* (interest rate parity; FX forward as a forward price under discounting)
- Neftci, *Principles of Financial Engineering* (FX market conventions; forward points and quoting)
- Hull, *Options, Futures, and Other Derivatives* (currency forward contracts; covered interest parity; hedging examples)
- Hull, *Risk Management and Financial Institutions* (forward valuation and replication interpretation)
- Oosterlee, *Mathematical Modeling and Computation in Finance* (FX forward valuation identities in discount-factor form)
- Cochrane, *Asset Pricing* (UIP and the forward discount/premium puzzle; forward vs expectation distinction)
- Taleb, *Dynamic Hedging* (forward/FX-forward delta intuition; discounting of the foreign leg)
