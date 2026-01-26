# Chapter 30: FX Swaps and Cross-Currency Swaps — Structure and Valuation Dependencies

---

## Introduction

An FX swap looks like two FX forwards—but it is really a funding instrument. This distinction explains why FX swaps constitute the world's largest derivatives market, with average daily turnover exceeding $3 trillion according to BIS surveys. When a European bank needs dollars to fund its U.S. operations, it does not necessarily borrow in the dollar market; instead, it might use an FX swap to convert euro funding into dollar funding at a locked-in exchange rate. When a Japanese pension fund hedges its U.S. Treasury holdings back to yen, it executes FX forwards or swaps that implicitly embed interest rate differentials between the two currencies.

Understanding these instruments requires keeping several related-but-distinct products straight. An **FX swap** combines a spot exchange with an opposite-direction forward—economically equivalent to borrowing one currency while lending another, with the exchange rate locked at both ends. The market is highly liquid at maturities out to one year. A **cross-currency swap** (XCCY) is a longer-dated instrument where two parties exchange floating-rate payments in different currencies, typically with notional exchanges at inception and maturity. Hull describes a currency swap as a contract where "two principals (one in each currency) are exchanged at the beginning and at the end of the swap's life." A **cross-currency basis swap** specifically refers to the floating-for-floating exchange where one leg pays a reference rate flat (say, USD SOFR) while the other pays its local reference rate plus or minus a spread—the **basis**. Andersen & Piterbarg define these as contracts "where floating Libor payments in one currency are exchanged for floating Libor payments in another currency, plus or minus a spread."

Why does the basis exist at all? Under textbook Covered Interest Parity (CIP), borrowing in one currency and converting to another should cost the same as borrowing directly in the target currency—any difference would create arbitrage. Yet in practice, persistent basis spreads of 20-50 basis points or more routinely appear. Andersen & Piterbarg document that "in the late 1990s, the CRX yield spread reached somewhere around -40 basis points in JPY as Japanese banks were perceived as being in economic trouble" and that "in early 2008 the CRX basis spread became significantly positive (up to +60 basis points) as hedging demands of long-dated FX books increased rapidly." These persistent deviations reflect credit differences between banking systems, balance sheet constraints that prevent arbitrage, and funding demand imbalances.

This chapter covers the **structure** of these instruments and the **valuation dependencies**—which curves price which legs, how FX conversion enters, and what risks emerge. We will:

1. Review FX forward mechanics and the discount-factor form of CIP (Section 30.1)
2. Define FX swaps and their funding interpretation (Section 30.2)
3. Define cross-currency swaps and the role of the basis spread (Section 30.3)
4. Develop the valuation framework: project in each currency, discount appropriately (Section 30.4)
5. Work through detailed examples including a 5-year EUR/USD xccy swap (Section 30.5)
6. Decompose risk exposures: FX delta, domestic/foreign PV01, basis DV01 (Section 30.6)
7. Discuss practical use cases: funding foreign assets, hedging foreign liabilities (Section 30.7)

**What this chapter does not cover:** The construction of cross-currency curves from market instruments belongs to Chapter 21. Full XVA treatment (CVA, FVA) and CSA mechanics are outside scope—we note only that collateralization affects which discount curve to use.

---

## Notation for This Chapter

| Symbol | Meaning |
|--------|---------|
| $d$ | Domestic currency (valuation/reporting currency) |
| $f$ | Foreign currency |
| $X(t)$ or $X(0)$ | Spot FX rate at time $t$, quoted as domestic per 1 unit of foreign (e.g., USD per EUR) |
| $F(0,T)$ | Forward FX rate for exchange at time $T$, same quote direction as $X$ |
| $P_d(0,T)$ | Domestic discount factor to $T$ (dimensionless) |
| $P_f(0,T)$ | Foreign discount factor to $T$ (dimensionless) |
| $P_c^{(L)}(0,T)$ | Projection curve for floating index in currency $c$ |
| $L_c(0, t_i, t_{i+1})$ | Forward floating rate for currency $c$ over $[t_i, t_{i+1}]$ |
| $\tau_i$ | Accrual year fraction for period $[t_i, t_{i+1}]$ |
| $N_d$, $N_f$ | Domestic and foreign notionals |
| $b$ (or $e$) | Cross-currency basis spread applied to one floating leg |

**Quote direction warning:** Some currency pairs are conventionally quoted inversely (e.g., "1 USD = x JPY" rather than "x USD per 1 JPY"). Always verify the quote direction before pricing.

---

## 30.1 FX Forwards and Covered Interest Parity

### 30.1.1 The Forward FX Rate

An FX forward contract sets a future exchange of currencies at a pre-agreed rate $F(0,T)$. Hull presents the fundamental relationship in his treatment of foreign currency forwards: the forward rate is pinned down by interest rate differentials, not by market expectations of future spot rates. In the continuous-compounding form that Hull provides:

$$\boxed{F_0 = S_0 e^{(r - r_f)T}}$$

where $S_0$ is the spot rate (domestic per foreign), $r$ is the domestic continuously compounded rate, and $r_f$ is the foreign rate. Hull explains the intuition: "A foreign currency has the property that the holder of the currency can earn interest at the risk-free interest rate prevailing in the foreign country... the foreign currency can be regarded as an asset providing a yield at a rate equal to the foreign risk-free interest rate."

Andersen & Piterbarg express the same relationship using discount factors, which avoids committing to a compounding convention and integrates naturally with curve-based valuation:

$$\boxed{F(0,T) = X(0) \frac{P_f(0,T)}{P_d(0,T)}} \tag{CIP-DF}$$

This is covered interest parity (CIP) in discount-factor form. The relationship arises because converting currencies via the forward market must produce the same value as converting via the spot market and investing the proceeds.

**Intuition:** If domestic interest rates exceed foreign rates ($P_d < P_f$ for the same maturity), then $F > X$—the foreign currency trades at a forward premium. This compensates a domestic investor for holding the lower-yielding foreign currency: the forward rate locks in an appreciation that offsets the interest differential.

**Unit check:** $P_f/P_d$ is dimensionless; $X(0)$ has units domestic/foreign; hence $F(0,T)$ has units domestic/foreign. ✓

### 30.1.2 Present Value of an FX Forward

Consider a forward where at time $T$ you receive 1 unit of foreign currency and pay $K$ units of domestic currency. The time-0 present value in domestic currency is:

$$\boxed{\text{PV}_0^{(d)} = X(0) P_f(0,T) - K P_d(0,T)} \tag{FXF-PV}$$

**Derivation:** "Receive 1 foreign at $T$" is economically equivalent to holding a foreign zero-coupon bond paying 1 at $T$, worth $P_f(0,T)$ in foreign terms; converting at spot gives domestic value $X(0) P_f(0,T)$. "Pay $K$ domestic at $T$" is a domestic ZCB liability of PV $K P_d(0,T)$. The net is their difference.

**At-market forward:** Setting $\text{PV}_0 = 0$ and solving for $K$:

$$K^* = X(0) \frac{P_f(0,T)}{P_d(0,T)} = F(0,T)$$

which confirms the CIP formula. Hull makes this point explicitly when valuing currency swaps: "Each exchange of payments in a fixed-for-fixed currency swap is a forward contract. As shown in Section 5.7, forward foreign exchange contracts can be valued by assuming that forward exchange rates are realized."

---

## 30.2 FX Swaps: The World's Largest Funding Market

### 30.2.1 Structure: Spot Plus Forward

An FX swap combines two legs:
- **Near leg:** Exchange notionals at spot rate $X(0)$
- **Far leg:** Reverse the exchange at a pre-agreed forward rate $F(0,T)$

For example, a bank with euros that needs dollars might enter the following transaction:

*Near leg (time 0):*
- Pay €100 million, receive $110 million at spot $X(0) = 1.10$ USD/EUR

*Far leg (time $T=1$):*
- Pay $111.5 million, receive €100 million at forward $F(0,1) = 1.115$

The bank has effectively borrowed dollars (received now, repaid later) and lent euros (paid now, received later), with the exchange rates locked.

> **Analogy: Renting Money**
>
> An FX Swap is essentially a **Collateralized Loan**.
>
> 1.  **You want to borrow Dollars**: But you don't have a U.S. credit history. You *do* have Euros.
> 2.  **The Deal**: You give me €100m Euros as "colateral". I give you \$110m Dollars.
> 3.  **The Return**: At the end of the year, we swap back.
> 4.  **The Interest**: Because I let you use my Dollars, and you let me use your Euros, the "interest" is handled by the **Forward Points** (the difference between the exchange rate today vs. next year).
>
> If you are absolutely desperate for Dollars, you might accept a terrible exchange rate for the swap back. That "desperation premium" is the Basis.

### 30.2.2 Why FX Swaps Dominate: The Funding Interpretation

The directional funding interpretation depends on which way the flows go:

| Structure | Funding Effect |
|-----------|----------------|
| Spot-buy foreign + Forward-sell foreign | Borrow foreign, lend domestic |
| Spot-sell foreign + Forward-buy foreign | Borrow domestic, lend foreign |

**Why this market is so large:** Banks, corporations, and asset managers constantly need to convert funding or hedging between currencies. An FX swap is cleaner than separate money-market transactions because the FX risk is eliminated (both legs are locked at known rates), the credit exposure is minimal (you are exchanging currencies, not lending unsecured), and documentation is standardized under ISDA.

Andersen & Piterbarg note that "the interbank FX forward market is rarely liquid beyond maturities of one year," which is why longer-dated cross-currency needs are typically met with cross-currency swaps rather than FX forwards or swaps.

### 30.2.3 FX Swap vs. FX Forward

A single-period FX swap where the near leg settles immediately is economically identical to an FX forward. Andersen & Piterbarg explicitly state that "a one-period CRX basis swap is identical to an FX forward contract." The difference is mainly in documentation and settlement mechanics—an FX swap typically involves actual exchange of notionals at inception, while a forward may be documented as a single future exchange.

---

## 30.3 Cross-Currency Swaps and the Basis

### 30.3.1 The Basic Structure

Hull describes the structure clearly: in a currency swap, "two principals (one in each currency) are exchanged at the beginning and at the end of the swap's life." In a **floating-for-floating** currency swap, each party pays a floating rate on its currency principal, with one leg possibly including a spread.

The canonical structure for a cross-currency basis swap:
- **At inception:** Exchange notionals at spot (e.g., pay \$100m USD, receive €90.9m at $X(0)=1.10$)
- **During the life:** Party A pays USD floating (e.g., SOFR); Party B pays EUR floating (e.g., EURIBOR) **plus or minus a spread** $b$
- **At maturity:** Re-exchange the same notionals

Hull notes that for floating-for-floating currency swaps: "A floating-for-floating swap can be valued by assuming that forward interest rates in each currency will be realized and discounting the cash flows at risk-free rates. The value of the swap is the difference between the values of the two sets of payments using current exchange rates."

### 30.3.2 The Cross-Currency Basis Spread

The basis spread $b$ is the key market observable that distinguishes cross-currency swaps from a simple combination of single-currency swaps. Andersen & Piterbarg measure the difference between projection and discount curves through what they call the "cross-currency (CRX) yield spread" $s(t)$, writing $P^{(L)}(t) = P(t) e^{-s(t)t}$.

**Sign conventions vary by market and desk.** The convention might be:
- "EUR leg pays EURIBOR + $b$" (basis quoted on non-USD leg)
- "USD leg pays SOFR + $b$" (less common)
- Positive $b$ meaning the non-USD borrower pays extra, or the reverse

I'm not sure a universal sign convention is specified in the sources. In practice, always confirm which leg carries the basis and the sign convention for your specific currency pair.

### 30.3.3 Why the Basis Exists: Funding Costs and Credit

Under textbook CIP, the basis should be zero—any non-zero basis would create arbitrage. Yet persistent non-zero bases are observed. Why?

Andersen & Piterbarg provide illuminating historical examples and explain the economics:

> "In the late 1990's, the CRX yield spread reached somewhere around -40 basis points in JPY as Japanese banks were perceived as being in economic trouble. During that period of time, foreign banks could generally fund themselves in USD at USD Libor, but in JPY at rates significantly below JPY Libor (due to their superior credit relative to Japanese banks)."

> "Conversely, in early 2008 the CRX basis spread became significantly positive (up to +60 basis points) as the hedging demands of long-dated FX books increased rapidly on the back of significant strengthening of the Yen versus the US Dollar."

**Economic drivers of the basis include:**

1. **Credit differences between banking systems:** If European banks are perceived as riskier than U.S. banks, European banks must pay more to access dollar funding—this shows up as a negative basis (they receive SOFR and pay EURIBOR minus some spread, netting to a cost).

2. **Balance sheet constraints:** Even when arbitrage appears profitable, banks face capital constraints, leverage ratio limits, and liquidity requirements that prevent them from scaling arbitrage to close the gap.

3. **Demand imbalances:** When many institutions simultaneously need dollar funding (e.g., Japanese insurers hedging U.S. assets back to yen), the basis adjusts to clear the market.

4. **Central bank interventions:** During the 2008 crisis and COVID-19, central bank swap lines helped alleviate dollar funding pressures, directly affecting the basis.

The persistence of the basis is not a market inefficiency in the usual sense—it reflects real frictions that cannot be arbitraged away by capital-constrained institutions.

> **The Crisis Indicator: The Vacuum Cleaner**
>
> A negative Cross-Currency Basis (e.g., EUR/USD Basis = -50bp) is one of the most widely watched "Fear Gauges" in global finance.
>
> *   **What it means**: Non-US banks are willing to pay a premium (receive *less* interest on their lending) just to get their hands on USD.
> *   **Why**: In a crisis (e.g., 2008, Covid 2020), the world rushes to buy USD assets (Safe Haven). Global banks desperately need USD to fund these purchases or roll over existing USD debt.
> *   **The Vacuum**: The demand sucks all the USD liquidity out of the system. The Basis widens (becomes more negative) as everyone tries to "Rent Dollars" at the same time.
>
> When the Basis blows out, Central Banks often step in with "Swap Lines" to flood the market with USD and normalize the spread.

---

## 30.4 Valuation Framework: Project and Discount in Each Currency

### 30.4.1 The Fundamental Principle

Hull provides the core valuation principle for floating-for-floating currency swaps: value each leg by assuming forward rates are realized, discount at each currency's risk-free rate, then convert at spot. Specifically:

> "A floating-for-floating swap can be valued by assuming that forward interest rates in each currency will be realized and discounting the cash flows at risk-free rates. The value of the swap is the difference between the values of the two sets of payments using current exchange rates."

Hull also shows how to value swaps using the bond approach: "If we define $V_{\text{swap}}$ as the value in U.S. dollars of an outstanding swap where dollars are received and a foreign currency is paid, then $V_{\text{swap}} = B_D - S_0 B_F$, where $B_F$ is the value, measured in the foreign currency, of the bond defined by the foreign cash flows on the swap and $B_D$ is the value of the bond defined by the domestic cash flows on the swap."

For a swap where you receive domestic floating and pay foreign floating plus basis:

$$\boxed{\text{PV}^{(d)} = \text{PV}_d^{(d)} - X(0) \cdot \text{PV}_f^{(f)}(b)}$$

where $\text{PV}_d^{(d)}$ is the domestic-currency PV of the domestic leg (discounted on the domestic curve), and $\text{PV}_f^{(f)}(b)$ is the foreign-currency PV of the foreign leg (discounted on the foreign curve, including the basis spread).

### 30.4.2 Separating Projection and Discounting

Modern multi-curve frameworks distinguish between:
- **Discount curves** $P_d$, $P_f$: Used to present-value known cashflows
- **Projection curves** $P_c^{(L)}$: Used to forecast floating rates via:

$$L_c(0, t_i, t_{i+1}) = \frac{1}{\tau_i}\left(\frac{P_c^{(L)}(0, t_i)}{P_c^{(L)}(0, t_{i+1})} - 1\right)$$

Andersen & Piterbarg explain this separation in detail. They introduce "the notion that when computing a swap value we may need two curves: i) the Libor 'pseudo-discount' curve $P^{(L)}(t)$, used to project the Libor-based floating cash flows on the floating leg of the swap; and ii) a real discount curve $P(t)$, used to discount all cash flows." They note that Libor rates "contain a certain amount of credit risk and it is ex-ante unclear that they are suitable proxies for a 'risk-free' rate."

### 30.4.3 The Full XCCY PV Formula

Andersen & Piterbarg provide an explicit formula for a USD/JPY basis swap where a USD-based party receives USD floating flat and pays JPY floating plus spread $e$, with notional \$1 exchanged for $X(0)$ yen at inception and re-exchanged at maturity:

$$\boxed{
\begin{aligned}
V_{\text{basisswap},\$}(0) &= \sum_{i=0}^{n-1} L_\$(0, t_i, t_{i+1}) \tau_i P_\$(0, t_{i+1}) + P_\$(0, t_n) \\
&\quad - X(0) \left( \sum_{i=0}^{n-1} (L_¥(0, t_i, t_{i+1}) + e) \tau_i P_¥(0, t_{i+1}) + P_¥(0, t_n) \right)
\end{aligned}
} \tag{XCCY-PV}$$

This formula appears as equation (6.41)-(6.42) in Andersen & Piterbarg. They simplify the USD leg using the fact that, under Assumption 6.5.1 where $P_\$$ and $P_\$^{(L)}$ are identical, "the time 0 price of the USD floating leg" reduces to \$1 (par value).

**What each curve does:**

| Curve | Role |
|-------|------|
| $P_\$(0,T)$ | Discounts USD cashflows |
| $P_¥(0,T)$ | Discounts JPY cashflows |
| $P_¥^{(L)}(0,T)$ | Generates forward JPY rates via ratio formula |
| $X(0)$ | Converts foreign-currency PV into domestic PV |
| $e^{\text{par}}$ | Market-quoted spread that sets $V = 0$ |

### 30.4.4 FX Conversion Consistency

Under CIP, there is an equivalence between two valuation approaches for a foreign cashflow $C_f(T)$:

**Approach A:** Foreign discount + spot conversion
$$\text{PV}^{(d)} = X(0) \cdot P_f(0,T) \cdot C_f(T)$$

**Approach B:** Forward FX + domestic discount
$$\text{PV}^{(d)} = P_d(0,T) \cdot F(0,T) \cdot C_f(T)$$

These are identical when $F(0,T) = X(0) \frac{P_f(0,T)}{P_d(0,T)}$:

$$P_d(0,T) \cdot F(0,T) = P_d(0,T) \cdot X(0) \frac{P_f(0,T)}{P_d(0,T)} = X(0) \cdot P_f(0,T)$$

**Why this matters:** If your curve construction is consistent (FX forwards implied from discount factor ratios), both approaches give the same answer. If not, you have an arbitrage inconsistency. Andersen & Piterbarg demonstrate this with a "zero-cost scheme" showing how to transform USD fixed cash flows into JPY fixed cash flows through a chain of swaps. They note: "If the JPY discount curve is inconsistent with the basis-swap market, the value computed this way may not equal the value computed by discounting the original USD cash flows at the USD discount curve. Since the swap transactions 1-3 above are costless, this discrepancy will indicate an arbitrage."

---

## 30.5 Worked Examples

### Example Conventions

Throughout these examples:
- Quote direction: $X =$ USD per EUR (domestic = USD, foreign = EUR)
- Rate conversion: Toy simple annual compounding $P(0,T) = 1/(1 + rT)$ (labeled as toy, not a market convention claim)
- Notional exchange: Included at maturity; initial exchange at spot has zero PV at inception

### 30.5.1 Example A: FX Swap Cashflows and Funding Interpretation

**Inputs:**
- Spot: $X(0) = 1.10$ USD/EUR
- Forward: $F(0,1) = 1.1150$ USD/EUR
- Notional: $N_f = €10{,}000{,}000$

**Structure:**

*Near leg (time 0):*
- Pay $N_d = X(0) N_f = 1.10 \times 10{,}000{,}000 = \$11{,}000{,}000$
- Receive €10,000,000

*Far leg (time $T=1$):*
- Pay €10,000,000
- Receive $F(0,1) N_f = 1.1150 \times 10{,}000{,}000 = \$11{,}150{,}000$

**Funding interpretation:**
- Pay USD now, receive USD later → **lending USD**
- Receive EUR now, pay EUR later → **borrowing EUR**
- The forward rate locks in the EUR repayment cost in USD terms

The implicit USD "interest" earned is $(\$11.15\text{m} - \$11.0\text{m})/\$11.0\text{m} = 1.36\%$, while the implicit EUR "interest" paid is locked through the exchange.

### 30.5.2 Example B: CIP Fair Forward and PV Calculation

**Inputs:**
- $X(0) = 1.10$ USD/EUR
- Domestic DF: $P_d(0,1) = 0.98$
- Foreign DF: $P_f(0,1) = 0.99$

**Step 1: Compute CIP fair forward**

$$F^*(0,1) = X(0) \frac{P_f(0,1)}{P_d(0,1)} = 1.10 \times \frac{0.99}{0.98} = 1.11122449 \text{ USD/EUR}$$

**Step 2: Compare to market forward**

Suppose the market quotes $F_{\text{mkt}}(0,1) = 1.1150$ USD/EUR. This is above the CIP-implied rate.

**Step 3: PV of receiving €10m and paying at market forward**

$$\text{PV}_0 = X(0) P_f(0,1) N_f - K P_d(0,1) N_f$$

Per EUR:
- $X(0) P_f = 1.10 \times 0.99 = 1.0890$
- $K P_d = 1.1150 \times 0.98 = 1.0927$
- Difference $= 1.0890 - 1.0927 = -0.0037$ USD/EUR

Total: $\text{PV}_0 = -0.0037 \times 10{,}000{,}000 = -\$37{,}000$

**Interpretation:** Paying a forward rate above the CIP-fair rate ($K > F^*$) makes this position negative PV.

### 30.5.3 Example C: 2-Period Cross-Currency Basis Swap Setup

**Inputs:**

| Parameter | Value |
|-----------|-------|
| Spot | $X(0) = 1.10$ USD/EUR |
| Payment dates | $t_1 = 0.5$, $t_2 = 1.0$; accruals $\tau_1 = \tau_2 = 0.5$ |
| USD discount factors | $P_d(0, 0.5) = 0.9900$, $P_d(0, 1.0) = 0.9800$ |
| EUR discount factors | $P_f(0, 0.5) = 0.9950$, $P_f(0, 1.0) = 0.9900$ |
| USD projection curve | $P_d^{(L)}(0, 0.5) = 0.9890$, $P_d^{(L)}(0, 1.0) = 0.9775$ |
| EUR projection curve | $P_f^{(L)}(0, 0.5) = 0.9940$, $P_f^{(L)}(0, 1.0) = 0.9880$ |
| Notionals | $N_d = \$100{,}000{,}000$, $N_f = N_d / X(0) = €90{,}909{,}090.91$ |

**Step 1: Compute forward floating rates**

Using $L = \frac{1}{\tau}\left(\frac{P^{(L)}(t_i)}{P^{(L)}(t_{i+1})} - 1\right)$:

*USD forwards:*
$$L_d(0; 0, 0.5) = \frac{1}{0.5}\left(\frac{1}{0.9890} - 1\right) = 2.2245\%$$
$$L_d(0; 0.5, 1.0) = \frac{1}{0.5}\left(\frac{0.9890}{0.9775} - 1\right) = 2.3529\%$$

*EUR forwards:*
$$L_f(0; 0, 0.5) = \frac{1}{0.5}\left(\frac{1}{0.9940} - 1\right) = 1.2072\%$$
$$L_f(0; 0.5, 1.0) = \frac{1}{0.5}\left(\frac{0.9940}{0.9880} - 1\right) = 1.2146\%$$

**Step 2: Cashflow schedule (receive USD float, pay EUR float + basis $b$)**

| Date | USD Leg (receive) | EUR Leg (pay) |
|------|-------------------|---------------|
| 0 | Pay \$100m, receive €90.91m | (opposite) |
| 0.5 | $100\text{m} \times 2.2245\% \times 0.5 = \$1{,}112{,}235$ | $90.91\text{m} \times (1.2072\% + b) \times 0.5$ |
| 1.0 | $100\text{m} \times 2.3529\% \times 0.5 = \$1{,}176{,}471$ | $90.91\text{m} \times (1.2146\% + b) \times 0.5$ |
| 1.0 | Receive \$100m principal | Pay €90.91m principal |

### 30.5.4 Example D: Solving for Par Basis Spread

We find $b_{\text{par}}$ such that the swap has zero PV at inception.

**Step 1: PV of USD leg**

- PV(coupon 0.5): $\$1{,}112{,}235 \times 0.99 = \$1{,}101{,}112$
- PV(coupon 1.0): $\$1{,}176{,}471 \times 0.98 = \$1{,}152{,}941$
- PV(principal): $\$100{,}000{,}000 \times 0.98 = \$98{,}000{,}000$

Total: $\text{PV}_d = \$100{,}254{,}053$

**Step 2: PV of EUR leg without basis**

Coupon amounts (no basis):
- At 0.5: $90{,}909{,}091 \times 1.2072\% \times 0.5 = €548{,}747$
- At 1.0: $90{,}909{,}091 \times 1.2146\% \times 0.5 = €552{,}080$

Discounted PV:
- PV(coupon 0.5): $€548{,}747 \times 0.9950 = €546{,}003$
- PV(coupon 1.0): $€552{,}080 \times 0.9900 = €546{,}559$
- PV(principal): $€90{,}909{,}091 \times 0.9900 = €90{,}000{,}000$

Total (no basis): $\text{PV}_{f,0} = €91{,}092{,}562$

Converted to USD: $X(0) \cdot \text{PV}_{f,0} = 1.10 \times €91{,}092{,}562 = \$100{,}201{,}818$

**Step 3: Basis PV term**

Basis adds $b \cdot \tau_i \cdot N_f$ at each payment. The PV of basis coupons:
$$\text{PV}_{\text{basis}}^{(f)} = N_f \cdot b \cdot \sum_{k=1}^{2} \tau_k P_f(0, t_k) = N_f \cdot b \cdot (0.5 \times 0.9950 + 0.5 \times 0.9900) = N_f \cdot b \times 0.9925$$

The "basis annuity": $A_f = €90{,}909{,}091 \times 0.9925 = €90{,}227{,}273$

**Step 4: Par condition**

$$0 = \text{PV}_d - X(0)(\text{PV}_{f,0} + A_f \cdot b_{\text{par}})$$

Solving:
$$b_{\text{par}} = \frac{\text{PV}_d / X(0) - \text{PV}_{f,0}}{A_f} = \frac{\$100{,}254{,}053 / 1.10 - €91{,}092{,}562}{€90{,}227{,}273}$$

$$b_{\text{par}} = \frac{€91{,}140{,}049 - €91{,}092{,}562}{€90{,}227{,}273} = \frac{€47{,}487}{€90{,}227{,}273} = 0.0526\% \approx 5.26 \text{ bp}$$

**Takeaway:** The par basis depends on spot FX, both discount curves, and both projection curves.

### 30.5.5 Example E: 5-Year EUR/USD Cross-Currency Swap with Basis

This example values a more realistic 5-year swap using the same framework.

**Inputs:**

| Parameter | Value |
|-----------|-------|
| Spot | $X(0) = 1.08$ USD/EUR |
| Notionals | \$100m USD, €92.59m EUR |
| Tenor | 5 years, annual payments |
| Basis | EUR leg pays EURIBOR + 25bp ($b = 0.0025$) |
| USD receives | SOFR flat |

**Discount and projection curves (illustrative):**

| Maturity | $P_d$ (USD) | $P_f$ (EUR) | $L_d$ (fwd SOFR) | $L_f$ (fwd EURIBOR) |
|----------|-------------|-------------|------------------|---------------------|
| 1Y | 0.9600 | 0.9750 | 4.00% | 2.50% |
| 2Y | 0.9180 | 0.9480 | 4.20% | 2.70% |
| 3Y | 0.8750 | 0.9200 | 4.30% | 2.90% |
| 4Y | 0.8320 | 0.8910 | 4.40% | 3.10% |
| 5Y | 0.7900 | 0.8620 | 4.50% | 3.25% |

**Step 1: USD leg cashflows and PV**

| Year | Forward Rate | Cashflow | DF | PV |
|------|--------------|----------|-----|-----|
| 1 | 4.00% | \$4,000,000 | 0.9600 | \$3,840,000 |
| 2 | 4.20% | \$4,200,000 | 0.9180 | \$3,855,600 |
| 3 | 4.30% | \$4,300,000 | 0.8750 | \$3,762,500 |
| 4 | 4.40% | \$4,400,000 | 0.8320 | \$3,660,800 |
| 5 | 4.50% | \$4,500,000 | 0.7900 | \$3,555,000 |
| 5 | Principal | \$100,000,000 | 0.7900 | \$79,000,000 |

**Total USD PV:** \$97,673,900

**Step 2: EUR leg cashflows and PV (including 25bp basis)**

| Year | Fwd Rate | Rate + Basis | Cashflow | DF | PV |
|------|----------|--------------|----------|-----|-----|
| 1 | 2.50% | 2.75% | €2,546,296 | 0.9750 | €2,482,639 |
| 2 | 2.70% | 2.95% | €2,731,481 | 0.9480 | €2,589,444 |
| 3 | 2.90% | 3.15% | €2,916,667 | 0.9200 | €2,683,333 |
| 4 | 3.10% | 3.35% | €3,101,852 | 0.8910 | €2,763,750 |
| 5 | 3.25% | 3.50% | €3,240,741 | 0.8620 | €2,793,519 |
| 5 | Principal | - | €92,592,593 | 0.8620 | €79,814,815 |

**Total EUR PV:** €93,127,500

**Step 3: Convert and compute swap PV**

$$\text{PV}^{(\$)} = \text{PV}_d - X(0) \cdot \text{PV}_f = \$97{,}673{,}900 - 1.08 \times €93{,}127{,}500$$

$$\text{PV}^{(\$)} = \$97{,}673{,}900 - \$100{,}577{,}700 = -\$2{,}903{,}800$$

**Interpretation:** At 25bp basis, this swap is off-market (negative PV for the USD receiver). The par basis would be lower than 25bp given these curves.

**Step 4: Compute par basis**

Using the basis annuity approach:
$$A_f = €92{,}592{,}593 \times (0.9750 + 0.9480 + 0.9200 + 0.8910 + 0.8620) = €92{,}592{,}593 \times 4.596 = €425{,}555{,}556$$

PV of EUR leg without basis ≈ €91,048,519 (removing the 25bp from each coupon)

$$b_{\text{par}} = \frac{\$97{,}673{,}900/1.08 - €91{,}048{,}519}{€425{,}555{,}556} = \frac{€90{,}438{,}796 - €91{,}048{,}519}{€425{,}555{,}556}$$

$$b_{\text{par}} \approx -14.3 \text{ bp}$$

The negative par basis indicates EUR funding is cheaper than USD funding in this curve environment—the EUR payer would receive a spread rather than pay one.

### 30.5.6 Example F: Hull-Style Currency Swap Valuation (Comparison)

This example follows Hull's Example 7.2 calculation style for comparison. Hull values a fixed-for-fixed currency swap using the bond approach.

**Inputs (from Hull Example 7.2):**
- USD rate: 2.5% continuously compounded
- JPY rate: 1.5% continuously compounded
- Spot: 110 yen per dollar (or 0.009091 USD/JPY)
- Swap: receive 3% JPY, pay 4% USD, principals ¥1,200m and \$10m

**Forward rates (per Hull's calculation):**
- 1Y forward: $0.009091 \times e^{(0.025-0.015) \times 1} = 0.009182$
- 2Y forward: $0.009091 \times e^{(0.025-0.015) \times 2} = 0.009275$
- 3Y forward: $0.009091 \times e^{(0.025-0.015) \times 3} = 0.009368$

Hull then values each annual exchange as a forward contract by "assuming that forward exchange rates are realized." The net value is the sum of forward contract values.

This approach—valuing each exchange of payments as a forward contract—works for all currency swap types, including floating-for-floating when combined with forward rate projection.

---

## 30.6 Risk Decomposition

### 30.6.1 FX Delta

For a position valued as $\text{PV}^{(d)} = \text{PV}_d - X(0) \cdot \text{PV}_f^{(f)}$, holding all curves and the foreign PV fixed:

$$\boxed{\frac{\partial \text{PV}^{(d)}}{\partial X(0)} = -\text{PV}_f^{(f)}}$$

**Interpretation:** If you pay the foreign leg, you are "short foreign PV" in domestic terms. When the foreign currency strengthens ($X$ rises for domestic/foreign quote), the foreign leg becomes more expensive, reducing your PV.

**From Example D:** FX delta $\approx -€91.14\text{m}$. A +1% spot move produces:
$$\Delta \text{PV} \approx -€91{,}140{,}049 \times 0.011 = -\$1{,}002{,}540$$

### 30.6.2 Domestic and Foreign PV01

**PV01** measures the PV change for a +1bp parallel shift in a yield curve. For cross-currency swaps:

- **Domestic PV01:** Bump domestic discount curve +1bp, holding foreign curves and FX fixed
- **Foreign PV01:** Bump foreign discount curve +1bp, holding domestic curves and FX fixed

**From Example D calculations:**
- Domestic PV01 ≈ -\$9,771 per bp (higher USD rates reduce USD cashflow PV)
- Foreign PV01 ≈ +\$9,893 per bp (higher EUR rates reduce what you owe, helping you)

**Important:** In a fully arbitrage-consistent framework, bumping one curve may require re-implying forwards. The above calculations hold FX forwards fixed, which is a common "risk decomposition" assumption but not the only valid choice. Andersen & Piterbarg note that "perturbations to funding instruments define sensitivities to discounting" while other perturbations capture basis risk.

### 30.6.3 Basis DV01

The basis spread $b$ enters additively in the foreign leg coupons. From the XCCY-PV formula:

$$\boxed{\frac{\partial \text{PV}}{\partial b} = -X(0) \cdot N_f \cdot \sum_i \tau_i P_f(0, t_{i+1})}$$

This is negative for the basis payer: when the basis you pay widens, your PV falls.

**From Example D:**
$$\text{Basis DV01} = -1.10 \times €90{,}909{,}091 \times 0.9925 / 10{,}000 \approx -\$9{,}925 \text{ per bp}$$

A +5bp widening costs approximately $5 \times \$9{,}925 = \$49{,}625$.

### 30.6.4 Risk Summary Table

For a receive-USD-float / pay-EUR-float-plus-basis position on \$100m:

| Risk Factor | Sensitivity | Sign Intuition |
|-------------|-------------|----------------|
| FX delta | -€91.14m | Short EUR: EUR strength hurts |
| USD PV01 | -\$9,771/bp | Long USD cashflows: higher rates hurt |
| EUR PV01 | +\$9,893/bp | Short EUR cashflows: higher EUR rates help |
| Basis DV01 | -\$9,925/bp | Paying basis: wider basis hurts |

---

## 30.7 Practical Applications

### 30.7.1 Funding Foreign Assets

**Scenario:** A U.S. insurance company holds €500m of European corporate bonds. The EUR-denominated coupons and principal create FX exposure back to the USD balance sheet.

**Solution:** Enter a cross-currency swap:
- Pay EUR floating + basis on €500m notional
- Receive USD floating on equivalent USD notional

**Effect:**
- EUR bond coupons approximately offset EUR floating payments
- USD floating receipts provide dollar cash flow
- FX exposure largely eliminated (notional exchange at maturity converts principal back)
- Residual exposures: basis spread risk, any mismatch between bond coupons and swap floating payments

### 30.7.2 Hedging Foreign Liabilities

**Scenario:** A German corporation has issued \$200m of USD-denominated bonds to access the deeper U.S. market. The USD coupons create FX exposure for a company whose revenues are in EUR.

**Solution:** Enter a cross-currency swap:
- Pay USD floating (to offset USD bond coupons, possibly with a USD fixed-to-float swap first)
- Receive EUR floating + basis

**Effect:**
- USD debt service transformed into EUR-linked payments
- Natural hedge against EUR revenues
- The basis spread determines the "all-in" EUR funding cost

### 30.7.3 Transforming the Currency of Funding

**Scenario:** A Japanese bank can borrow cheaply in JPY but needs USD to fund its U.S. lending operations.

**Solution:**
1. Borrow in JPY at domestic rates
2. Enter FX swap or cross-currency swap to convert to USD

**Economics:**
- The bank effectively borrows at USD rates plus/minus the cross-currency basis
- If JPY/USD basis is negative (Japanese institutions pay to get USD), the USD funding cost exceeds raw USD rates
- This is precisely why the basis market exists—it prices the relative cost of currency-transformed funding

---

## 30.8 Practical Notes and Common Pitfalls

### 30.8.1 Convention Checklist

| Item | What to Verify |
|------|----------------|
| **FX quote direction** | USD/EUR vs EUR/USD; forward points vs outright quotes |
| **Which leg gets basis** | Non-USD leg vs USD leg; check confirmations |
| **Sign convention** | Positive basis = payer pays extra, or receiver receives less? |
| **Notional exchange timing** | At inception, maturity, or both? Resettable notionals? |
| **Collateral currency** | CSA terms determine discount curve choice |

### 30.8.2 Common Errors

1. **Mixing projection and discount curves:** The projection curve generates forward rates; the discount curve PVs cashflows. Do not use one for both. Andersen & Piterbarg emphasize that "when computing a swap value we may need two curves."

2. **Inconsistent FX conversion:** If you convert foreign cashflows using forwards, ensure $F(0,T) = X(0) P_f(0,T)/P_d(0,T)$ holds—or acknowledge you are accepting a "model inconsistency" for risk decomposition.

3. **Treating basis as "just a spread":** The basis embeds curve dependencies. Changing basis quotes requires consistent curve re-bootstrapping, not just adding a spread.

4. **Confusing FX swap and XCCY swap:** FX swaps are short-dated funding instruments (< 1 year); XCCY swaps are long-dated (1-30 years) with different risk profiles.

### 30.8.3 Implementation Pitfalls

1. **Calendar misalignment:** Payment dates may not align across currencies. A robust date engine handling multiple holiday calendars is essential.

2. **Interpolation artifacts:** If implied forward FX is computed from interpolated discount factors, ensure the FX curve is smooth and consistent with the basis curve.

3. **Bump-and-rebuild vs. bump-fixed:** When computing PV01, decide whether to hold other curves/FX fixed or re-bootstrap consistently. Document the choice. As Andersen & Piterbarg note, the parameterization should "naturally aggregate 'similar' risks such as overall rate level risks, discounting risks, basis risks, while keeping different kinds separate for efficient risk management."

---

## Summary

1. **FX forwards link spot and interest rates via CIP:** $F(0,T) = X(0) \cdot P_f(0,T)/P_d(0,T)$

2. **FX swaps are funding instruments:** Spot exchange + forward reversal = synthetic cross-currency borrowing/lending

3. **FX swaps dominate short maturities** (< 1 year) due to liquidity; longer tenors use cross-currency swaps

4. **Cross-currency swaps exchange floating legs** in two currencies, typically with notional exchange at inception and maturity

5. **The basis spread compensates for funding/credit differentials** between currency money markets—not arbitrageable away due to capital constraints

6. **Modern valuation separates projection and discounting:** Project floating rates from $P^{(L)}$, discount with $P$, convert at spot FX

7. **Hull's valuation principle:** "Assume forward rates are realized and discount cash flows at risk-free rates"

8. **Andersen & Piterbarg's XCCY-PV formula** makes curve dependencies explicit: domestic discount, foreign discount, projection curves, spot FX, basis spread

9. **Key risks:** FX delta (short foreign PV), domestic PV01, foreign PV01, basis DV01—each requires specifying what's held fixed

10. **Practical uses:** Funding foreign assets, hedging foreign liabilities, transforming funding currency

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| FX Forward | Agreement to exchange currencies at future date at agreed rate | Building block for FX swaps; prices embed interest differentials |
| CIP (Discount Factor Form) | $F = X \cdot P_f/P_d$ | Links FX forwards to yield curves; deviations signal funding stress |
| FX Swap | Spot + opposite forward exchange | Largest derivatives market; primary tool for cross-currency funding |
| Cross-Currency Swap | Exchange of floating payments in two currencies with notional exchange | Transforms currency and rate basis of assets/liabilities |
| Cross-Currency Basis | Spread added to one floating leg to make swap par | Prices relative funding costs between currency markets |
| Projection Curve | Curve used to forecast floating rates | Distinct from discount curve in multi-curve framework |
| Basis DV01 | PV change per 1bp basis move | Key risk for XCCY positions; often hardest to hedge |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | Write CIP in discount-factor form | $F(0,T) = X(0) \cdot P_f(0,T)/P_d(0,T)$ |
| 2 | Write Hull's continuous-compounding CIP formula | $F_0 = S_0 e^{(r-r_f)T}$ |
| 3 | What does an FX swap combine? | Spot FX exchange + opposite-direction forward exchange |
| 4 | Funding interpretation: spot-buy foreign + forward-sell foreign | Borrow foreign currency, lend domestic currency |
| 5 | Why are FX swaps the largest derivatives market? | Primary tool for cross-currency funding; clean credit exposure |
| 6 | What's the typical maturity limit for liquid FX forwards? | About 1 year (per Andersen & Piterbarg) |
| 7 | Define cross-currency basis swap | Exchange floating rates in two currencies, one leg plus/minus a spread |
| 8 | Why does the basis exist if CIP should make it zero? | Credit differences, balance sheet constraints, demand imbalances |
| 9 | PV of FX forward (receive 1 foreign, pay K domestic at T) | $X(0) P_f(0,T) - K P_d(0,T)$ |
| 10 | What role does the projection curve play? | Forecasts forward floating rates via discount factor ratios |
| 11 | What role does the discount curve play? | Present-values known cashflows |
| 12 | FX delta formula for $\text{PV} = \text{PV}_d - X \cdot \text{PV}_f$ | $\partial \text{PV}/\partial X = -\text{PV}_f$ |
| 13 | Are you long or short foreign currency when paying the foreign leg? | Short foreign (FX appreciation hurts you) |
| 14 | Basis DV01 sign for paying the basis | Negative (wider basis reduces your PV) |
| 15 | How do you convert foreign PV to domestic under CIP? | Multiply by spot: $\text{PV}^{(d)} = X(0) \cdot \text{PV}_f^{(f)}$ |
| 16 | Alternative conversion using forwards | $\text{PV}^{(d)} = P_d(0,T) \cdot F(0,T) \cdot C_f(T)$ |
| 17 | When are the two conversion methods equivalent? | When CIP holds: $P_d \cdot F = X \cdot P_f$ |
| 18 | What happened to JPY basis in late 1990s per A&P? | Reached approximately -40bp |
| 19 | What does negative basis mean for EUR leg payer? | EUR payer receives spread (EUR funding relatively cheaper) |
| 20 | What's "par basis"? | Spread that makes swap PV zero at inception |

---

## Mini Problem Set

### Questions with Solution Sketches (1-8)

**Q1:** Given $X(0) = 1.25$, $P_d(0,1) = 0.96$, $P_f(0,1) = 0.98$, compute the no-arbitrage forward.

*Sketch:* $F = 1.25 \times 0.98/0.96 = 1.2760$ domestic/foreign.

---

**Q2:** You have an FX forward to receive €1m and pay \$1.30m at T=1. Given $X(0)=1.28$, $P_d=0.97$, $P_f=0.98$, compute PV.

*Sketch:* $\text{PV} = 1.28 \times 0.98 \times 1{,}000{,}000 - 1.30 \times 0.97 \times 1{,}000{,}000 = 1{,}254{,}400 - 1{,}261{,}000 = -\$6{,}600$

---

**Q3:** Show that $P_d(0,T) \cdot F(0,T) = X(0) \cdot P_f(0,T)$ algebraically using CIP-DF.

*Sketch:* Substitute $F = X \cdot P_f/P_d$ into LHS: $P_d \cdot X \cdot P_f/P_d = X \cdot P_f = $ RHS.

---

**Q4:** Explain why inconsistent basis-swap inputs create arbitrage per Andersen & Piterbarg.

*Sketch:* Their zero-cost scheme chains trades: (1) swap fixed USD to USD Libor+x, (2) swap USD Libor+x to JPY Libor+e+x in CRX basis swap, (3) swap JPY Libor+e+x to fixed JPY. If the resulting JPY cashflows have different PV than direct USD→JPY conversion at spot, you have risk-free profit.

---

**Q5:** A 3-year EUR leg pays EURIBOR + 15bp. Given $N_f = €50\text{m}$, annual payments, and $P_f(0,1)=0.98$, $P_f(0,2)=0.95$, $P_f(0,3)=0.91$, compute the PV of the basis payments in EUR.

*Sketch:* Basis PV $= 50\text{m} \times 0.0015 \times (0.98 + 0.95 + 0.91) = 50\text{m} \times 0.0015 \times 2.84 = €213{,}000$

---

**Q6:** Derive that basis DV01 = $-X(0) \cdot N_f \cdot \sum \tau_i P_f(0, t_{i+1})$ for the basis payer.

*Sketch:* The basis term adds $b \cdot \tau_i \cdot N_f$ at each date, discounted at $P_f$. Total basis PV $= N_f \cdot b \cdot \sum \tau_i P_f$. Convert to domestic: $X(0) \cdot N_f \cdot b \cdot \sum \tau_i P_f$. Subtract (paying) and differentiate by $b$.

---

**Q7:** For $\text{PV} = \text{PV}_d - X \cdot \text{PV}_f$, what is the FX delta if $\text{PV}_f^{(f)} = €80\text{m}$?

*Sketch:* FX delta $= -€80\text{m}$. A +1% FX move (if $X$ goes from 1.10 to 1.111) costs $€80\text{m} \times 0.011 = \$880{,}000$.

---

**Q8:** Why is "domestic PV01" not uniquely defined without specifying what's held fixed?

*Sketch:* In arbitrage-consistent curves, bumping domestic rates may require re-implying FX forwards or foreign curves. Different "hold fixed" assumptions give different PV01s.

---

### Questions without Solutions (9-16)

**Q9:** Compute par basis for a 3-year semi-annual XCCY swap given specific curve inputs.

**Q10:** Show how basis DV01 sign flips when you switch from pay-basis to receive-basis.

**Q11:** Explain how XCCY basis swaps help build long-dated cross-currency curves when FX forwards are illiquid.

**Q12:** Design a hedge portfolio that neutralizes FX delta and USD PV01 while leaving basis exposure.

**Q13:** How does collateral currency affect discount curve choice? What additional data is needed?

**Q14:** Design a "CIP-consistent" bump scheme where domestic curve bump automatically adjusts FX forwards.

**Q15:** Compare valuing a foreign bond via (i) foreign discount + spot, (ii) domestic discount + forwards. Under what assumptions are they identical?

**Q16:** Create a P&L attribution template separating FX, domestic curve, foreign curve, and basis. State limitations.

---

## Source Map

### (A) Verified Facts — Source-Backed

| Fact | Source |
|------|--------|
| FX forward parity $F_0 = S_0 e^{(r-r_f)T}$ | Hull, *Options, Futures, and Other Derivatives*, Ch 5, Eq 5.9 |
| "A foreign currency has the property that the holder can earn interest at the risk-free rate prevailing in the foreign country" | Hull, Ch 5 |
| Forward FX in discount factor form $F = X \cdot P_f/P_d$ | Andersen & Piterbarg, Vol 1, Ch 6.5.2, Eq 6.38 |
| Currency swap structure: "two principals (one in each currency) are exchanged at the beginning and at the end" | Hull, Ch 7.8 |
| Floating-for-floating swap valuation: "assume forward interest rates in each currency will be realized and discount at risk-free rates" | Hull, Ch 7.10 |
| Bond approach: $V_{\text{swap}} = B_D - S_0 B_F$ | Hull, Ch 7.9 |
| Cross-currency basis swap definition: "floating Libor payments in one currency are exchanged for floating Libor payments in another currency, plus or minus a spread" | Andersen & Piterbarg, Vol 1, Ch 6.5.2.3 |
| Forward rate from projection curve | Andersen & Piterbarg, Vol 1, Ch 6.5.2.2, Eq 6.40 |
| One-period basis swap = FX forward | Andersen & Piterbarg, Vol 1, Ch 6.5.2.3 |
| FX forwards rarely liquid beyond 1 year | Andersen & Piterbarg, Vol 1, Ch 6.5.2.3 |
| Collateralized discounting ties to OIS | Andersen & Piterbarg, Vol 1, Ch 6.5.3 |
| CRX spread reached -40bp JPY late 1990s | Andersen & Piterbarg, Vol 1, Ch 6.5.2.2 |
| CRX spread reached +60bp early 2008 | Andersen & Piterbarg, Vol 1, Ch 6.5.2.2 |
| XCCY-PV formula with explicit curve roles | Andersen & Piterbarg, Vol 1, Ch 6.5.2.3, Eq 6.41-6.42 |
| Separation of projection and discount curves for swap valuation | Andersen & Piterbarg, Vol 1, Ch 6.5.2.2 |
| Zero-cost scheme demonstrating arbitrage from inconsistent curves | Andersen & Piterbarg, Vol 1, Ch 6.5.2.3 |
| Hull Example 7.2: currency swap valuation with forward rates | Hull, Ch 7.9 |

### (B) Reasoned Inference — Derived

| Inference | Derivation |
|-----------|------------|
| CIP at $t=0$ in DF form | Specialization of A&P forward formula |
| FX forward PV formula | Replication with ZCBs: long foreign bond + short domestic bond |
| FX delta = $-\text{PV}_f^{(f)}$ | Differentiation of PV formula $\text{PV} = \text{PV}_d - X \cdot \text{PV}_f$ with respect to $X$ |
| Basis DV01 linear structure | Differentiation of XCCY-PV by basis spread $b$ |
| Par basis formula | Setting PV = 0 and solving for $b$ |
| Equivalence of two FX conversion methods | Algebraic substitution of CIP formula |

### (C) Flagged Uncertainties

| Uncertainty | What Would Be Needed |
|-------------|----------------------|
| Universal sign convention for basis | Market convention documentation for specific currency pair; ISDA definitions |
| XCCY variants (resettable notionals, mark-to-market, etc.) | Product standard documentation for specific structures |
| Multi-currency collateral discounting | CSA terms, collateral currency, remuneration rate for specific trade |
| Calendar/rolling conventions for specific currency pairs | Product documentation, ISDA schedule |
| Exact spot settlement conventions | Market practice varies (T+2 for most, T+1 for some); sources do not specify |
