# Chapter 21: Cross-Currency Curves — CIP, FX Forwards, and Cross-Currency Basis as Curve Constraints

---

## Introduction

A trader in London wants to fund a USD position using EUR. She could borrow EUR from her treasury desk, convert to USD at spot, invest the dollars, and hedge the FX exposure with a forward—a textbook covered interest arbitrage. In the pre-2008 world, this transaction would yield exactly the same return as borrowing USD directly, because *covered interest parity* (CIP) held with machine-like precision. The forward FX rate would adjust to eliminate any profit from the round-trip.

That world ended in August 2007. As the credit crisis unfolded, the CIP relationship—one of the most fundamental no-arbitrage conditions in finance—began to break down. Banks suddenly found that funding USD through FX markets cost more than borrowing USD directly, even after accounting for interest rate differentials. This "cross-currency basis" widened from negligible levels to dozens of basis points, and it has never fully returned to zero. Andersen and Piterbarg document episodes where the cross-currency yield spread reached "around -40 basis points in JPY" during the late 1990s Japanese banking crisis, and "significantly positive (up to +60 basis points)" in early 2008 as hedging demands surged.

The emergence of persistent CIP deviations has profound implications for curve construction. **If you build USD and JPY discount curves independently from their respective local swap markets—without imposing consistency with FX forwards—you will create arbitrageable inconsistencies.** As Andersen and Piterbarg emphasize, "the market for foreign exchange (FX) forwards and cross-currency basis swaps imposes certain arbitrage constraints that must be considered in the curve construction exercise."

> **Analogy: The Teleporter**
>
> Imagine you want 100 EUR in one year. You have two ways to get it:
> 1.  **The Local Path**: Wait in Europe. (Invest EUR today).
> 2.  **The Teleporter**: Wait in the US, then teleport. (Invest USD today, then swap to EUR).
>
> **Covered Interest Parity (CIP)** says these two paths must cost exactly the same. The FX Forward market is the "Teleporter." If the Teleporter is cheaper than the Local Path, everyone will use it, forcing the price back in line.
>
> **The Crisis**: In 2008, the Teleporter broke. It became much more expensive to use the Teleporter (swap USD for EUR) than to borrow locally, because banks stopped trusting the "teleportation" mechanism (counterparty risk).

This chapter develops the machinery for cross-currency curve construction. We will cover:

1. **Covered Interest Parity** (Section 21.1): The no-arbitrage relationship linking spot FX, forward FX, and discount factors across currencies—the foundation for everything that follows.
2. **FX Forwards as Curve Constraints** (Section 21.2): How forward quotes, combined with a domestic curve, pin down the foreign discount curve.
3. **Cross-Currency Basis** (Section 21.3): What "basis" means at the curve level, why it exists, why it persists despite "arbitrage," and what drives it (including issuance dynamics and the USD premium).
4. **Cross-Currency Basis Swaps** (Section 21.4): The instruments that provide constraint information beyond the short-dated FX forward market, including structural details and cashflow mechanics.
5. **Multi-Currency Curve Construction** (Section 21.5): A practical algorithm for building consistent cross-currency curves, with a complete numerical walkthrough.
6. **Collateral Currency and Curve Choice** (Section 21.6): How the currency of posted collateral affects which discount curve applies—essential for production systems.
7. **Risk Decomposition** (Section 21.7): FX delta, rates PV01, and basis exposure as distinct risk dimensions, including P&L attribution frameworks.

**Chapter boundaries:** This chapter focuses on the *curve construction implications* of cross-currency markets. Chapter 29 covers FX forward mechanics and pricing in detail; Chapter 30 covers cross-currency swap structures and valuation. Here, we treat these instruments primarily as sources of constraint equations that must be satisfied when building curves. Chapter 20 develops the multi-curve framework for a single currency (tenor basis); this chapter extends that framework to multiple currencies.

---

## 21.1 Covered Interest Parity (CIP): The Foundation

Covered interest parity is the no-arbitrage relationship that links spot FX, forward FX, and interest rates across two currencies. It is "covered" because the FX exposure is hedged with a forward contract, eliminating currency risk from the arbitrage argument. Hull calls this "the well-known interest rate parity relationship from international finance."

### 21.1.1 CIP in Rate Form

Hull presents the currency forward pricing identity under continuous compounding. He explains that "a foreign currency can be regarded as an investment asset paying a known yield," where the yield is the foreign risk-free rate. Following this logic:

$$\boxed{F_0 = S_0 \, e^{(r_d - r_f)T}}$$

where $S_0$ is the spot exchange rate (domestic currency per unit of foreign), $r_d$ is the domestic risk-free rate, $r_f$ is the foreign risk-free rate, and $T$ is the time to maturity.

**Replication argument.** Hull presents this elegantly in his textbook. Consider a firm with 1,000 units of domestic currency wanting exposure to foreign currency at time $T$:

- **Strategy A**: Buy foreign currency spot, invest at the foreign rate, ending with $1000/S_0 \times e^{r_f T}$ units of foreign currency.
- **Strategy B**: Invest domestically at $r_d$, convert at time $T$ using a forward at rate $F_0$, ending with $(1000 \times e^{r_d T})/F_0$ units of foreign currency.

As Hull states: "In the absence of arbitrage opportunities, the two strategies must give the same result." Hence:

$$\frac{1000 \times e^{r_f T}}{S_0} = \frac{1000 \times e^{r_d T}}{F_0}$$

Rearranging yields the CIP formula.

> **The "Free Lunch" Calculation**
>
> Hull provides a concrete example of this "Free Lunch":
> "Suppose that the 2-year interest rates in Australia and the United States are 3% and 1%, respectively, and the spot exchange rate is 0.7500 USD per AUD."
>
> From the CIP formula, the 2-year forward exchange rate should be:
> $$F_0 = 0.7500 \times e^{(0.01-0.03) \times 2} = 0.7500 \times e^{-0.04} = 0.7206$$
>
> **The Glitch**: Suppose the market forward is 0.7000 (too low).
> 1.  **Borrow**: Borrow 1,000 AUD at 3%. (You owe 1,061.84 AUD in 2 years).
> 2.  **Convert**: Sell 1,000 AUD for 750 USD.
> 3.  **Invest**: Invest 750 USD at 1%. (You get 765.15 USD in 2 years).
> 4.  **Lock**: Enter a Forward to BUY 1,061.84 AUD at 0.7000. This will cost you $743.29 USD.
> 5.  **Profit**: You have $765.15 USD. You pay $743.29 USD to pay off your loan.
> 6.  **Result**: You keep **$21.86 USD**. Risk-free. From thin air.

### 21.1.2 CIP in Discount-Factor Form

For curve construction, the discount-factor formulation is more useful because it avoids specifying compounding conventions. Andersen and Piterbarg derive the forward FX rate via a replication using zero-coupon bonds.

Consider a forward contract to receive 1 unit of foreign currency at time $T$ in exchange for $K$ units of domestic currency. They establish the fundamental constraint:

$$\boxed{F(0,T) = X(0) \frac{P_f(0,T)}{P_d(0,T)}}$$

where $X(0)$ is the spot FX rate (domestic per foreign), and $P_d, P_f$ are the domestic and foreign discount factors.

**Derivation.** Consider a forward contract to receive 1 unit of foreign currency at time $T$ in exchange for $K$ units of domestic currency. As Andersen and Piterbarg explain, the time-0 present value of each leg is:

- **Foreign leg (receive)**: The domestic PV of 1 foreign at $T$ is $X(0) P_f(0,T)$
- **Domestic leg (pay)**: The domestic PV of paying $K$ at $T$ is $K P_d(0,T)$

Setting PV = 0 at inception (the definition of a forward rate):

$$X(0) P_f(0,T) - K P_d(0,T) = 0 \quad \Rightarrow \quad K = X(0) \frac{P_f(0,T)}{P_d(0,T)} = F(0,T)$$

Andersen and Piterbarg present this as equation (6.38), noting that the forward FX rate $X_T(t)$ is given by:

$$X_T(t) = X(t) \frac{P_f(t,T)}{P_d(t,T)}$$

and they explain: "The name is motivated by the following arbitrage strategy: Buy one foreign zero-coupon bond, at a cost of $\widetilde{P}_d(t,T)$ in domestic currency. Finance the purchase by selling short domestic zero-coupon bonds... With no outlay at time $t$, the strategy will generate a net cash flow at time $T$ of one unit of foreign currency and $-X_T(t)$ units of domestic currency."

**Unit check.** $P_f/P_d$ is dimensionless (ratio of discount factors). Multiplying by $X(0)$ (domestic/foreign) gives domestic per foreign—matching the forward quote. ✓

**Sanity check.** If $P_d = P_f$ (equal discounting in both currencies), then $F = X$: the forward equals spot when there is no interest rate differential. ✓

### 21.1.3 Reconciling the Two Forms

The rate and discount-factor forms are equivalent under continuous compounding. If $P_d(0,T) = e^{-r_d T}$ and $P_f(0,T) = e^{-r_f T}$:

$$\frac{P_f(0,T)}{P_d(0,T)} = \frac{e^{-r_f T}}{e^{-r_d T}} = e^{(r_d - r_f)T}$$

Plugging into the DF-form CIP recovers Hull's rate-form formula. The discount-factor form is preferred in practice because it avoids specifying compounding conventions and handles term structures naturally.

---

## 21.2 FX Forwards as Curve Constraints

The CIP relationship transforms FX forwards from tradable instruments into *constraint equations* for curve construction. Given knowledge of the domestic discount curve and the spot/forward FX market, we can derive the implied foreign discount curve.

### 21.2.1 The Fundamental Constraint Equation

Rearranging the CIP equation, Andersen and Piterbarg (equation 6.38) present the cross-currency arbitrage constraint:

$$\boxed{P_d(T) = X(0) P_f(T) Y(T) \quad \Rightarrow \quad P_f(T) = \frac{P_d(T)}{Y(T) X(0)}}$$

where $Y(T)$ is the forward FX rate in the *inverse* quote direction (foreign per domestic). In our standard notation (forward as domestic per foreign):

$$\boxed{P_f(0,T) = \frac{F(0,T)}{X(0)} P_d(0,T)}$$

This is the "FX-implied foreign discount factor" formula. It tells us that given an observed FX forward and a domestic discount curve, the foreign discount factor is uniquely determined.

### 21.2.2 What Happens When Curves Are Built Independently

Andersen and Piterbarg explicitly warn about the consequences of ignoring this constraint. In their own words:

> "Suppose, say, that we have blindly estimated discount curves $P(\cdot)$ and $P_¥(\cdot)$ from the market for USD- and JPY-denominated interest rate swaps, respectively, without paying any attention to FX markets. The discount curves estimated in this fashion will very likely not satisfy [the CIP constraint], implying the existence of cross-currency arbitrages. The degree to which [the constraint] is typically violated is often small, but any such violation can be highly problematic for a firm engaging in trading of significant amounts of both USD- and JPY-denominated assets."

The violation creates the following exploit: a trader could use the three-step conversion illustrated by Andersen and Piterbarg:

1. **Swap fixed USD → USD LIBOR** in a regular USD interest rate swap
2. **Swap USD LIBOR → JPY LIBOR** in a cross-currency basis swap
3. **Swap JPY LIBOR → fixed JPY** in a regular JPY interest rate swap

As they explain: "If the JPY discount curve is inconsistent with the basis-swap market, the value computed this way may not equal the value computed by discounting the original USD cash flows at the USD discount curve. Since the swap transactions 1-3 above are costless, this discrepancy will indicate an arbitrage."

### 21.2.3 Building an FX-Implied Foreign Curve

Given a strip of FX forwards and a domestic discount curve, we can construct the implied foreign discount curve point by point.

> **Example 21.2: FX-Implied EUR Curve from USD Curve**
>
> **Given:**
> - Domestic = USD, Foreign = EUR
> - Spot $X(0) = 1.1000$ USD/EUR
> - USD discount curve: $P_d(0,0.5) = 0.985112$, $P_d(0,1) = 0.970446$, $P_d(0,2) = 0.941765$
> - Observed FX forwards: $F(0,0.5) = 1.111055$, $F(0,1) = 1.122221$, $F(0,2) = 1.144892$
>
> **Step 1: Implied EUR discount factors**
>
> Using $P_f(0,T) = \frac{F(0,T)}{X(0)} P_d(0,T)$:
>
> | $T$ | $F(0,T)/X(0)$ | $P_d(0,T)$ | $P_f(0,T)$ |
> |-----|---------------|------------|------------|
> | 0.5 | 1.010050 | 0.985112 | 0.995012 |
> | 1.0 | 1.020201 | 0.970446 | 0.990050 |
> | 2.0 | 1.040811 | 0.941765 | 0.980199 |
>
> **Step 2: Implied EUR zero rates** (continuous compounding)
>
> $r_f(T) = -\frac{1}{T} \ln P_f(0,T)$
>
> | $T$ | $P_f(0,T)$ | $r_f(T)$ |
> |-----|------------|----------|
> | 0.5 | 0.9950 | 1.00% |
> | 1.0 | 0.9901 | 1.00% |
> | 2.0 | 0.9802 | 1.00% |
>
> **Sanity checks:**
> - Discount factors decreasing with maturity ✓
> - Implied foreign rate (1%) less than domestic rate (3%), consistent with forward premium ✓
> - Forward ratio matches rate differential: $e^{0.02 \times 2} = 1.0408 \approx 1.040811$ ✓

### 21.2.4 The Triangular Arbitrage Logic

The constraint relationship can be visualized as a *triangle* connecting three objects: domestic curve, foreign curve, and FX forwards. Given any two, the third is determined. Inconsistency means the triangle doesn't close.

```
          Domestic Curve (P_d)
                 /\
                /  \
               /    \
              /      \
             /        \
    FX Forwards -------- Foreign Curve (P_f)
         (F)

Constraint: F = X · (P_f / P_d)
```

> **The Triangle Check**
>
> 1.  **Spot Market**: Current Exchange.
> 2.  **Interest Rates**: Cost of Time (Domestic vs Foreign).
> 3.  **Forward Market**: Future Exchange.
>
> If you know any two, you *must* satisfy the third. If your curve construction uses Independent Domestic Swaps and Independent Foreign Swaps, it will almost certainly fail to match the Forward Market, breaking the triangle.
```

In multi-currency trading, all three sides of this triangle are liquid markets. The curve construction problem is to ensure the triangle closes exactly—or, more realistically, to decide which market to treat as the "anchor" and derive the others.

---

## 21.3 Cross-Currency Basis: What It Is and Why It Exists

In a frictionless world, the FX-implied foreign curve would exactly match the curve derived from local foreign-currency swaps. In practice, they differ. This difference is the **cross-currency basis**.

### 21.3.1 Defining the Basis at the Curve Level

Andersen and Piterbarg introduce a *pseudo-discount curve* $P^{(L)}$ for LIBOR projections, separate from the true discount curve $P$. The difference is captured by a yield spread:

$$\boxed{P^{(L)}(t) = P(t) \, e^{-s(t) \cdot t}}$$

Solving for the spread:

$$s(t) = -\frac{1}{t} \ln\left(\frac{P^{(L)}(t)}{P(t)}\right)$$

Andersen and Piterbarg call $s(t)$ the "cross-currency (CRX) yield spread."

The cross-currency basis represents the cost (or benefit) of synthesizing funding in one currency by borrowing in another currency and swapping. When the basis is negative, it costs more to obtain foreign currency funding via the FX market than to borrow directly in that foreign currency. When positive, the opposite is true.

### 21.3.2 Why the Basis Exists: Post-Crisis Reality

Before 2007, the basis was negligible—typically a fraction of a basis point. Andersen and Piterbarg note that "the spread between the Fed funds rate and 3 month Libor rate used to be very small—in the order of a few basis points—after September 2007 it went up to as much as 275 basis points."

The crisis revealed fundamental market segmentation:

1. **Credit risk differentiation**: Banks in different countries have different credit quality. A Japanese bank might fund in JPY at JPY LIBOR, but a foreign bank could fund in JPY at rates *below* JPY LIBOR due to superior credit. This creates a wedge between local-market rates and FX-implied rates.

2. **Balance sheet constraints**: Post-crisis regulations (Basel III leverage ratios, capital requirements) make it expensive for banks to warehouse FX risk, even for arbitrage. The trade that "should" compress the basis requires balance sheet capacity that banks are unwilling to deploy.

3. **Funding segmentation**: Different investor bases access different markets. U.S. money market funds might demand dollar assets while Japanese institutions seek yen. These preferences create supply-demand imbalances that translate into basis.

4. **Hedging demand imbalances**: Andersen and Piterbarg note that in early 2008, "the hedging demands of long-dated FX books increased rapidly on the back of significant strengthening of the Yen versus the US Dollar," driving the CRX basis spread to +60 basis points.

### 21.3.3 Historical Episodes

Andersen and Piterbarg document two striking examples that illustrate the dramatic moves that can occur:

**Late 1990s JPY (negative basis):** "The CRX yield spread reached somewhere around -40 basis points in JPY as Japanese banks were perceived as being in economic trouble. During that period of time, foreign banks could generally fund themselves in USD at USD Libor, but in JPY at rates significantly below JPY Libor (due to their superior credit relative to Japanese banks)."

**Early 2008 JPY (positive basis):** "In early 2008 the CRX basis spread became significantly positive (up to +60 basis points) as the hedging demands of long-dated FX books increased rapidly on the back of significant strengthening of the Yen versus the US Dollar."

**2007-2009 crisis:** "Many other currencies (including EUR) have experienced similar dramatic moves in the CRX basis spreads against USD."

These episodes demonstrate that the basis is not a theoretical curiosity but a material risk factor that can dominate P&L.

### 21.3.4 Why CIP "Arbitrage" Doesn't Eliminate the Basis

In textbook arbitrage, exploiting a mispricing eliminates it. Why doesn't this happen with CIP deviations?

1. **Capital requirements**: The "arbitrage" requires holding positions that consume regulatory capital. The cost of that capital may exceed the arbitrage profit.

2. **Counterparty credit limits**: The trades require counterparty exposure across currencies. Banks may not have sufficient credit lines.

3. **Term funding constraints**: Long-dated CIP deviations require long-dated funding, which may be unavailable or expensive.

4. **Risk limits**: Even "riskless" arbitrage consumes risk limits (VaR, DV01 limits) that banks may not want to use.

The basis represents a *shadow price* of these constraints—the cost of converting funding across currencies. In effect, the limits on arbitrage capacity are themselves priced into the basis.

> **Desk Reality: The Balance Sheet Economics of "Arbitrage"**
>
> Consider a bank trying to exploit a 30bp CIP deviation in 5-year EUR/USD. The textbook trade is:
> 1. Borrow EUR for 5 years
> 2. Swap to USD via xccy basis swap
> 3. Invest USD for 5 years
>
> **The problem**: This trade requires *funded balance sheet*. Under Basel III:
> - **Leverage ratio**: The notional appears in the denominator. A $1bn trade might consume 3-5% of a bank's leverage ratio capacity.
> - **Risk-weighted assets**: Even with minimal credit risk, standardized RWA applies.
> - **Liquidity coverage ratio**: Term funding creates LCR drag.
>
> A bank with a 15% return-on-equity target and 4% leverage ratio capacity cost might need 15% × 4% = 60bp+ return to justify the trade—more than the 30bp basis profit.
>
> **This is why basis persists**: The "arbitrage" costs more in balance sheet terms than it earns in basis terms.

### 21.3.5 The USD Premium: Why Non-US Banks Pay for Dollars

> **Practitioner Note:** The following section extends beyond the textbook sources to explain the structural market dynamics driving cross-currency basis. This is essential desk-level knowledge for understanding basis direction and magnitude.

The cross-currency basis is often described as "the price of USD liquidity." Understanding why requires grasping the structural imbalances in global dollar funding.

**Why non-US banks need USD:**

1. **Asset-liability mismatch**: Japanese, European, and other non-US banks hold substantial USD assets (corporate loans, securities) funded by local currency deposits. They need to *swap* their domestic funding into USD.

2. **Dollar dominance**: The USD is the global reserve currency. Trade finance, commodity transactions, and international lending all require USD.

3. **No USD lender of last resort**: Non-US banks cannot access the Fed's discount window. In a crisis, their USD funding can evaporate while domestic central banks can only provide local currency.

**The supply-demand imbalance:**

| Side | Who | What They Do | Effect on Basis |
|------|-----|--------------|-----------------|
| **Demand** | Non-US banks | Need USD, have EUR/JPY/etc | Pay spread to swap into USD |
| **Supply** | US money funds, corporates | Have USD, limited need for foreign CCY | Receive spread to supply USD |

When demand exceeds supply, the non-USD leg pays a negative spread (by market convention). This is the "USD premium"—the rental cost for borrowing dollar balance sheet.

> **Desk Reality: Quarter-End Basis Widening**
>
> Every quarter-end, and especially year-end, basis widens predictably. Why?
>
> **Regulatory window-dressing**: Banks' leverage ratios are measured on reporting dates. To meet targets, banks shrink balance sheets at quarter-end, withdrawing from xccy markets. With less supply, basis widens.
>
> **The pattern**:
> - Basis tightens in the first 2 months of each quarter
> - Basis widens in the final 2-3 weeks before quarter-end
> - Spikes on the actual reporting date
> - Snaps back immediately after
>
> Traders who understand this seasonality can position accordingly—or at minimum, avoid getting caught paying wide basis into quarter-end.

**Central bank swap lines as circuit breakers:**

During the 2008 crisis and again in March 2020, the Federal Reserve established swap lines with major central banks (ECB, BoJ, BoE, SNB). These allow foreign central banks to borrow USD from the Fed and lend to their domestic banks.

When swap lines are activated at scale, they act as a *ceiling* on basis—banks can always get USD from their central bank (at a penalty rate) rather than paying extreme basis in the market. The 2020 COVID crisis saw basis widen dramatically before swap line deployment, then compress as the Fed provided unlimited USD liquidity.

> **Historical Episode: March 2020 COVID Basis Blow-Out**
>
> In early March 2020, EUR/USD 3-month basis traded around -20bp (normal for the period). Over two weeks:
> - March 9-13: Basis widened to -50bp as pandemic fears grew
> - March 16-19: Basis blew out to -150bp as dollar funding seized
> - March 19: Fed announced unlimited swap lines with major central banks
> - March 23-31: Basis compressed back to -30bp
>
> The episode demonstrated: (1) basis risk is real and can dominate P&L in stress; (2) central bank intervention can rapidly reverse moves; (3) even "arbitrage-free" relationships break when balance sheets seize.

**Sign convention clarification:**

The basis spread is typically quoted on the non-USD leg. Convention:
- **Negative basis** (e.g., -30bp on EUR leg): EUR-funded party pays to swap into USD
- **Positive basis** (e.g., +20bp on EUR leg): USD-funded party pays to swap into EUR

Most of the time since 2008, non-USD currencies have traded with negative basis against USD—reflecting the structural USD shortage.

### 21.3.6 Trading the Basis: Issuance and Funding Flows

> **Practitioner Note:** This section describes real-world market flows that drive basis. While not covered in the standard textbooks, this knowledge is essential for understanding basis as an equilibrium price.

**Who trades basis and why?**

The cross-currency basis equilibrates supply and demand for currency-specific funding. Understanding the participants helps predict basis direction.

| Participant | Typical Position | Why | Typical Size |
|-------------|-----------------|-----|--------------|
| **Corporate issuers** | Receive USD funding via swap | Issue in EUR (low rates), swap to USD | $bn single deals |
| **Non-US bank ALM** | Pay for USD funding | Fund USD assets with domestic liabilities | Ongoing, structural |
| **Central banks** | Swap reserves into USD | Diversify FX reserves | Very large, lumpy |
| **Japanese lifers** | Hedge USD assets | Buy US Treasuries, hedge FX risk | Seasonal (fiscal year) |
| **Relative value desks** | Trade around fair value | Capture basis moves vs. model | Opportunistic |
| **Asset managers** | Hedge foreign holdings | International equity/bond funds | Index-driven |

**Issuance Arbitrage: The Corporate Treasury Perspective**

> **Desk Reality: How Apple Funds Itself in EUR and Swaps to USD**
>
> Consider a hypothetical US corporate (like Apple) that can issue:
> - USD bonds at Treasury + 80bp = 4.80% (example)
> - EUR bonds at Bunds + 60bp = 3.60% (example)
>
> With EUR/USD basis at -30bp (EUR leg), the swap works as follows:
>
> **Step 1:** Issue €500mm 5-year bonds at 3.60%
>
> **Step 2:** Enter 5Y EUR/USD xccy swap:
> - Receive EUR SOFR equivalent (to pay EUR bondholders)
> - Pay USD SOFR + basis adjustment
> - At inception: receive €500mm, pay $550mm (at spot)
> - At maturity: pay €500mm, receive $550mm
>
> **Step 3:** Net effect: All-in USD funding cost
>
> The math (simplified):
> - EUR spread paid: 60bp
> - EUR swap rate: ~3.00% (hypothetical)
> - USD swap rate: ~4.00% (hypothetical)
> - Basis adjustment: -30bp
>
> All-in USD cost ≈ 4.00% + 60bp - 30bp = 4.30%
>
> **vs. direct USD issuance**: 4.80%
>
> **Savings**: 50bp! On $550mm for 5 years, this is material.
>
> **Why this works**: The corporate exploits its *credit spread* being tighter in EUR than in USD (60bp vs 80bp), and the basis *paying it* to swap. High-quality corporates can access EUR investors who are starved for yield.

**What issuance tells us about basis direction:**

When many corporates issue in EUR and swap to USD:
- **Supply effect**: More EUR flowing into the xccy market (selling EUR basis)
- **Basis tightens**: EUR basis becomes less negative (moves toward zero)

When issuance dries up:
- **Demand effect**: Bank ALM demand still needs USD
- **Basis widens**: EUR basis becomes more negative

This creates a feedback loop: wide basis incentivizes issuance arbitrage, which compresses basis.

**The "Yankee Reverse": How Foreign Banks Fund USD Directly**

Japanese, European, and other non-US banks can reduce their dependence on xccy swaps by issuing USD-denominated instruments directly:

- **USD CDs and commercial paper**: Short-term funding from US money market funds
- **Yankee bonds**: Longer-term USD bonds sold to US investors
- **USD repo**: Secured borrowing against USD collateral

When foreign banks can access these markets cheaply, they need fewer xccy swaps, reducing demand and compressing basis. When these markets seize (as in 2008, 2020), banks must turn to xccy swaps, widening basis.

> **Example 21.4b: All-In Funding Cost Calculation**
>
> A European bank needs $1bn USD funding for 3 months. Options:
>
> **Option A: Swap from EUR**
> - EUR deposit rate: 3.50% (EURIBOR)
> - 3M EUR/USD basis: -40bp
> - All-in USD cost: USD SOFR + (-40bp) basis = SOFR - 40bp? No!
>
> **Correction**: The basis is quoted on the EUR leg. The bank *pays* EUR + basis to receive USD flat.
> - If basis is -40bp on EUR, bank receives EUR EURIBOR - 40bp vs. paying USD SOFR flat
> - Net cost: SOFR flat, but synthetically funded
>
> **Option B: Issue 3M USD CD**
> - If the bank can issue at SOFR + 20bp, this is cheaper than swapping at wide basis
>
> **Option C: Borrow from central bank swap line (crisis only)**
> - Fed-ECB swap line: OIS + 25bp (penalty rate)
> - Only available when lines are activated

---

## 21.4 Cross-Currency Basis Swaps: Constraints Beyond FX Forwards

FX forwards are typically liquid only out to about one year. For longer maturities, the constraint information comes from **cross-currency basis swaps**.

### 21.4.1 Structure of a Cross-Currency Basis Swap

Andersen and Piterbarg provide a clear description: "CRX basis swaps are contracts where floating Libor payments in one currency are exchanged for floating Libor payments in another currency, plus or minus a spread. The swaps involve an exchange of notionals at trade inception and at maturity; the ratio between the two notionals is normally set to equal the spot FX exchange rate prevailing at trade inception."

**Key structural features:**

| Feature | Description |
|---------|-------------|
| **Floating legs** | Each currency pays its local LIBOR (or successor rate) |
| **Basis spread** | Added to one leg (convention varies) |
| **Notional exchange** | At inception and maturity, at spot FX ratio |
| **Maturities** | Quoted out to 30+ years |

**Critical insight:** Andersen and Piterbarg emphasize that "a one-period CRX basis swap is identical to an FX forward contract." This establishes the continuity between short-dated FX forwards and long-dated basis swaps. In fact, the basis swap generalizes the FX forward to multiple periods with intermediate floating payments.

> **The Plumbing Diagram: Cashflows of a 5Y EUR/USD Xccy Basis Swap**
>
> Party A (USD-based) receives EUR EURIBOR + basis, pays USD SOFR flat
>
> ```
> INCEPTION (t=0):
> ─────────────────────────────────────────────────────────
>   Party A ──────────────> €100mm ──────────────> Party B
>   Party A <──────────────$110mm <──────────────Party B
>                          (at spot 1.10)
>
> QUARTERLY PAYMENTS (t=0.25, 0.50, 0.75, ... 4.75):
> ─────────────────────────────────────────────────────────
>   Party A ──────> SOFR × τ × $110mm ──────> Party B
>   Party A <──── (EURIBOR + e) × τ × €100mm <──── Party B
>                  (e = basis spread, could be -30bp)
>
> MATURITY (t=5):
> ─────────────────────────────────────────────────────────
>   Party A ──────────────> $110mm ──────────────> Party B
>   Party A <──────────────€100mm <──────────────Party B
>                          (at ORIGINAL spot rate)
>   Plus final coupon exchange
> ```
>
> **Key observations:**
> 1. **Principal exchange at spot, repaid at same rate**: FX risk on the notional exchange is locked in at inception
> 2. **Floating-for-floating with basis**: Unlike IRS, both legs float
> 3. **Basis on non-USD leg**: If e = -30bp, EUR receiver gets EURIBOR - 30bp
> 4. **No FX risk on coupons during life**: Each coupon stays in its own currency

**Mark-to-Market (MTM) vs. Resettable Structures:**

> **Practitioner Note:** In modern markets, many long-dated xccy swaps use "mark-to-market" or "resettable" structures to reduce counterparty exposure.

In a standard xccy swap, the FX rate is locked at inception. If spot moves significantly, the PV mismatch creates large counterparty exposure.

**MTM xccy swap**: The notional exchange rate resets periodically (typically annually) to the prevailing spot rate. This reduces PFE (potential future exposure) and counterparty risk, but introduces additional cashflows to settle the notional revaluation.

**I'm not sure** about the exact mechanics of MTM swaps across all markets—conventions vary by currency pair and counterparty relationship.

### 21.4.2 Valuation of a Cross-Currency Basis Swap

For a USD/JPY basis swap where a USD-based corporation receives USD LIBOR flat in exchange for JPY LIBOR plus spread $e_¥$, Andersen and Piterbarg give the valuation formula (equation 6.42):

$$V_{\text{basisswap},\$}(0) = 1 - X(0) \times \left(\sum_{i=0}^{n-1}\left(\frac{P_¥^{(L)}(0,t_i)}{P_¥^{(L)}(0,t_{i+1})} - 1 + e_¥ \tau_i\right) P_¥(0,t_{i+1}) + P_¥(0,t_n)\right)$$

where:
- The USD floating leg plus notional repayment collapses to par (\$1) under the assumption that USD index = USD discount curve
- The JPY leg is valued using forward JPY LIBOR rates from $P_¥^{(L)}$ and discounted at $P_¥$

As they explain: "The market quotes par values $e_¥^{par}$—that is, the value of $e_¥$ that will make $V_{\text{basisswap},\$}(0) = 0$—in a wide range of maturities extending out to 30 years or more."

### 21.4.3 Why Basis Swaps Provide Curve Constraints

Failing to fit basis swaps creates the same arbitrage as failing to fit FX forwards. Andersen and Piterbarg illustrate with the three-step conversion:

1. Swap fixed USD → USD LIBOR (regular USD IRS)
2. Swap USD LIBOR → JPY LIBOR + $e$ (CRX basis swap)
3. Swap JPY LIBOR + $e$ → fixed JPY (regular JPY IRS)

They conclude: "If the JPY discount curve is inconsistent with the basis-swap market, the value computed this way may not equal the value computed by discounting the original USD cash flows at the USD discount curve. Since the swap transactions 1-3 above are costless, this discrepancy will indicate an arbitrage."

> **Example 21.3: Solving for Par Basis Spread**
>
> **Setup:** 2-period toy basis swap (domestic USD, foreign JPY)
> - Spot $X(0) = 0.0091$ USD/JPY
> - Payment dates: $t_1 = 0.5$, $t_2 = 1.0$; accrual $\tau = 0.5$
> - Foreign discount curve: $P_f(0,0.5) = 0.9950$, $P_f(0,1) = 0.9900$
> - Foreign index curve: $P_f^{(L)}(0,0.5) = 0.9945$, $P_f^{(L)}(0,1) = 0.9885$
> - USD floating leg = par (simplifying assumption)
>
> **Step 1: Compute forward JPY LIBOR rates**
>
> $L_0 = \frac{1}{0.5}\left(\frac{1}{0.9945} - 1\right) \approx 1.106\%$
>
> $L_1 = \frac{1}{0.5}\left(\frac{0.9945}{0.9885} - 1\right) \approx 1.214\%$
>
> **Step 2: Write par condition**
>
> $1 = (L_0 + e) \cdot 0.5 \cdot P_{0.5} + (L_1 + e) \cdot 0.5 \cdot P_1 + P_1$
>
> **Step 3: Compute "no-basis" floating leg PV**
>
> Interest PV: $L_0 \cdot 0.5 \cdot 0.9950 + L_1 \cdot 0.5 \cdot 0.9900 = 0.00550 + 0.00601 = 0.01151$
>
> Total (including principal): $0.01151 + 0.9900 = 1.00151$
>
> **Step 4: Solve for par spread**
>
> Annuity: $A = 0.5 \times 0.9950 + 0.5 \times 0.9900 = 0.9925$
>
> $1 = 1.00151 + e \times 0.9925$
>
> $e = \frac{1 - 1.00151}{0.9925} \approx -15.2 \text{ bp}$
>
> **Interpretation:** The negative spread on the JPY leg compensates for the fact that JPY forward rates (projected from the index curve) exceed the "risk-free equivalent" implied by pure CIP. This is the basis.

---

## 21.5 Multi-Currency Curve Construction: A Practical Algorithm

### 21.5.1 The Degrees of Freedom Problem

With two currencies, we have potentially four curves:
- $P_d(t)$: domestic discount curve
- $P_d^{(L)}(t)$: domestic index curve
- $P_f(t)$: foreign discount curve
- $P_f^{(L)}(t)$: foreign index curve

But we have only three independent market sources:
- Domestic swaps (fixing $P_d^{(L)}$)
- Foreign swaps (fixing $P_f^{(L)}$)
- FX forwards / basis swaps (linking the currencies)

Andersen and Piterbarg observe: "It should be clear that the introduction of the pseudo-discount curves $P_\$^{(L)}(t)$ and $P_¥^{(L)}(t)$ equips us with enough degrees of freedom to fit both USD-denominated swaps, JPY-denominated swaps, and the market for FX forward contracts. In fact, we have too many degrees of freedom: four curves, but only three separate markets to calibrate to."

**Resolution via "bedrock" assumption:** Andersen and Piterbarg introduce Assumption 6.5.1: "In USD, the Libor pseudo-discount curve coincides with the real discount curve." This anchors the system by setting $P_\$^{(L)} = P_\$$.

Post-crisis, this assumption is replaced by using OIS for discounting, with the Libor-OIS spread determined from domestic basis swaps (see Chapter 20). As Andersen and Piterbarg note: "it is now generally accepted that the Libor rate is no longer a good proxy for a discounting rate on collateralized trades."

### 21.5.2 The Iterative Algorithm

Andersen and Piterbarg outline a modified curve construction procedure:

**Step 1: Build USD curves**

Using standard bootstrapping from:
- Funding instruments (deposits, OIS)
- LIBOR-referencing instruments (FRAs, swaps)

This gives $P_\$(t)$ and $P_\$^{(L)}(t)$.

**Step 2: Build foreign index curve $P_f^{(L)}(t)$**

From foreign-currency swaps, bootstrap $P_f^{(L)}(t)$ as if it were a discount curve.

**Step 3: Calibrate foreign discount curve $P_f(t)$ from basis swaps**

Represent $P_f$ as a multiplicative spread to $P_f^{(L)}$:

$$P_f(t) = P_f(T_i) \frac{P_f^{(L)}(t) e^{-\varepsilon_i(t - T_i)}}{P_f^{(L)}(T_i)}, \quad t \in [T_i, T_{i+1})$$

where $\varepsilon_i$ is a piecewise constant spread intensity.

As Andersen and Piterbarg explain: "The instantaneous forward rates generated by $P_¥(t)$ are given by those computed from $P_¥^{(L)}(t)$ plus a piecewise flat function":

$$f_¥(t) = f_¥^{(L)}(t) + \varepsilon(t), \quad \varepsilon(t) = \sum_{i=0}^{N-1} \varepsilon_i \mathbf{1}_{\{t \in [T_i, T_{i+1})\}}$$

The constants $\varepsilon_0, \ldots, \varepsilon_{N-1}$ are calibrated by fitting to par-valued CRX basis swaps maturing at $T_1, \ldots, T_N$.

**Step 4: Iterate if necessary**

The initial bootstrap of $P_f^{(L)}$ treated it as a discount curve. With $P_f$ now calibrated, we can re-price the foreign benchmark instruments using the correct multi-curve valuation. Iterate steps 2-3 until convergence.

Andersen and Piterbarg describe the iteration: "The iteration is initiated... with the estimate $\mathbf{V}^{(L)}(0) = \mathbf{V}$ and runs until the termination criterion is satisfied. As the approximation $\mathbf{V}^{(L)} \approx \mathbf{V}$ is normally very accurate, only a few iterations are needed to reach acceptable precision."

### 21.5.3 Numerical Walkthrough: The Iterative Algorithm in Action

> **Example 21.5: Complete Iterative Curve Construction**
>
> We build a simplified 2-point EUR discount curve using the Andersen-Piterbarg algorithm.
>
> **Given:**
> - USD OIS curve (discount = index by bedrock assumption):
>   - $P_\$(0,1) = 0.9600$, $P_\$(0,2) = 0.9200$
> - Spot EUR/USD: $X(0) = 1.10$ USD/EUR
> - EUR swap rates: 1Y @ 3.00%, 2Y @ 3.20% (annual, 30/360)
> - EUR/USD basis swaps: 1Y @ -20bp, 2Y @ -30bp (on EUR leg)
>
> **Step 1: Initialize with $V^{(L)}(0) = V$**
>
> EUR swap prices assuming par: $V = (1.00, 1.00)$
>
> **Step 2: Build initial EUR index curve $P_{EUR}^{(L)}$**
>
> From 1Y swap at 3.00%: $(0.03)(1) P_{EUR}^{(L)}(1) + 1 \cdot P_{EUR}^{(L)}(1) = 1.00$
> $$P_{EUR}^{(L)}(0,1) = \frac{1.00}{1.03} = 0.9709$$
>
> From 2Y swap at 3.20%: $(0.032) P_{EUR}^{(L)}(1) + (1.032) P_{EUR}^{(L)}(2) = 1.00$
> $$P_{EUR}^{(L)}(0,2) = \frac{1.00 - 0.032 \times 0.9709}{1.032} = 0.9383$$
>
> **Step 3: Calibrate EUR discount curve to basis swaps**
>
> Using equation (6.44)-(6.45), we parameterize:
> $$P_{EUR}(T) = P_{EUR}^{(L)}(T) \cdot e^{-\varepsilon(T) \cdot T}$$
>
> For 1Y basis swap at -20bp ($e = -0.0020$):
>
> Par condition (simplified): FX-implied constraint must hold
> $$P_{EUR}(0,1) = \frac{F(0,1)}{X(0)} P_\$(0,1)$$
>
> With basis adjustment: the effective forward embeds the basis spread.
>
> Solving iteratively: $\varepsilon_0 \approx 0.0018$ (18bp spread intensity)
>
> $P_{EUR}(0,1) = 0.9709 \times e^{-0.0018 \times 1} = 0.9691$
>
> For 2Y: $\varepsilon_1 \approx 0.0028$ (28bp spread intensity)
>
> $P_{EUR}(0,2) = 0.9383 \times e^{-0.0028 \times 2} = 0.9331$
>
> **Step 4: Re-price EUR swaps with multi-curve valuation**
>
> Using $P_{EUR}^{(L)}$ for projection and $P_{EUR}$ for discounting:
>
> 1Y EUR swap value:
> - Fixed leg: $0.03 \times P_{EUR}(1) = 0.03 \times 0.9691 = 0.02907$
> - Floating leg: Forward rate × DF = same by construction
> - Value: $V^{(1)}(1) \approx 0.9998$ (close to 1.00)
>
> 2Y EUR swap value: $V^{(1)}(2) \approx 0.9996$
>
> **Step 5: Check convergence**
>
> $|V^{(1)} - V| = |(0.9998, 0.9996) - (1.00, 1.00)| < 0.001$
>
> Tolerance met? If not, update: $V^{(L)}(1) = V^{(L)}(0) - (V^{(1)} - V)$
>
> **Step 6: Iterate until convergence**
>
> | Iteration | $P_{EUR}^{(L)}(2)$ | $P_{EUR}(2)$ | Swap PV Error |
> |-----------|-------------------|--------------|---------------|
> | 0 | 0.9383 | 0.9331 | 0.0004 |
> | 1 | 0.9385 | 0.9333 | 0.00005 |
> | 2 | 0.9385 | 0.9333 | < 0.00001 ✓ |
>
> **Converged in 2-3 iterations** — as Andersen notes, "the approximation $V^{(L)} \approx V$ is normally very accurate, only a few iterations are needed."

> **Desk Reality: Why Iteration Converges Quickly**
>
> The initial guess ($P^{(L)} = P$, i.e., index = discount) is already close to the truth in most cases. The basis spreads are typically small (tens of basis points), so the correction is minor.
>
> When basis is large (crisis periods), more iterations may be needed, but the algorithm remains stable because we're making incremental adjustments to a fundamentally sound approximation.

### 21.5.4 Avoiding Circularity

A subtle issue arises: if we use cross-currency basis swaps to infer the discounting basis in USD, we create circular reasoning—we're using non-USD markets to determine USD discounting.

Andersen and Piterbarg's resolution: "To estimate this basis in USD, we need to rely on domestic markets only; doing otherwise will introduce a circularity into our arguments."

In practice, the USD Libor-OIS basis is calibrated from:
- OIS swaps (overnight index swaps)
- Fed Funds / LIBOR basis swaps

Then cross-currency instruments translate this basis into other currencies. The key insight is that the USD curve must be self-consistent before extending to other currencies.

### 21.5.5 The Modern 3-Curve Framework

Post-LIBOR transition, the standard framework for EUR/USD is:

| Curve | Source Instruments | Role |
|-------|-------------------|------|
| **USD OIS** | SOFR swaps | USD discounting + USD projection |
| **EUR OIS** | ESTR swaps | EUR discounting |
| **EUR/USD basis** | Cross-currency basis swaps | Links EUR OIS to USD OIS |

The algorithm:
1. Build USD OIS from SOFR swaps
2. Build EUR OIS from ESTR swaps (using EUR OIS as projection)
3. Calibrate EUR discount curve adjustments from xccy basis swaps
4. Iterate until consistent

---

## 21.6 Collateral Currency and Curve Choice

The discussion so far has assumed standard collateral practices. In reality, the currency in which collateral is posted affects which discount curve applies. Andersen and Piterbarg note that "uncollateralized derivative contracts are subject to credit risk, and a fully consistent pricing approach needs to incorporate the cost of hedging this risk."

### 21.6.1 The Collateral-Discount Curve Linkage

The fundamental principle: **discount at the collateral rate**.

For a fully collateralized swap where collateral earns the overnight rate:
- **USD collateral** → discount at USD OIS (SOFR)
- **EUR collateral** → discount at EUR OIS (ESTR)
- **Multi-currency CSA** → more complex (optionality)

This linkage arises because the posted collateral earns the overnight rate. The counterparty receiving collateral funds the position at the same rate, making the overnight rate the appropriate discount rate.

### 21.6.2 What If Collateral Currency Differs from Trade Currency?

> **Example 21.6: Same Swap, Different Collateral, Different Value**
>
> Consider a 5Y USD interest rate swap (pay fixed, receive SOFR):
> - Notional: $100mm
> - Fixed rate: 4.00%
> - Current USD OIS rate: 4.20%
>
> **Case A: USD collateral (standard)**
>
> Discount at USD OIS. PV of fixed leg uses USD OIS DFs.
>
> Approximate PV: $-\sum_{t=1}^{5} (0.04 - 0.042) \times DF_{USD}(t) \times 100mm$
>
> With average DF ≈ 0.90: PV ≈ $-0.002 \times 0.90 \times 5 \times 100mm = -\$900,000$
>
> **Case B: EUR collateral**
>
> If collateral is posted in EUR, the discount rate should be the EUR "equivalent" rate—which incorporates the xccy basis.
>
> EUR OIS rate: 3.00%
> EUR/USD basis: -30bp
>
> Effective USD discount rate: USD OIS - basis adjustment
>
> With EUR collateral, the USD swap effectively discounts at a rate ~30bp lower than pure USD OIS (because EUR collateral costs less to fund).
>
> This changes the PV by approximately:
> $$\Delta PV \approx 0.0030 \times \text{Duration} \times \text{Notional} = 0.0030 \times 4.5 \times 100mm = \$1.35mm$$
>
> **The swap is worth more under EUR collateral** when EUR funding is cheaper.
>
> **Case C: No collateral (uncollateralized)**
>
> Discount at the counterparty's funding rate + credit adjustments (CVA/DVA). This is significantly more complex and beyond simple curve construction.

### 21.6.3 Practical Implications

| Collateral Scenario | Discount Curve | Who Cares |
|--------------------|----------------|-----------|
| USD CSA, USD trade | USD OIS | Standard case |
| EUR CSA, USD trade | USD OIS adjusted for basis | Large xccy books |
| Choice of collateral currency | Optionality value | Sophisticated counterparties |
| No collateral | Funding + CVA/DVA | Credit desks |

**I'm not sure** about the exact mechanics of multi-currency CSA optionality pricing—this requires modeling the option to post in the cheapest currency, which involves basis volatility assumptions.

> **Desk Reality: Why CSA Currency Matters**
>
> A European bank trading USD swaps with a US counterparty might have different valuations depending on CSA terms:
> - **USD CSA**: Standard Bloomberg/ISDA valuation applies
> - **EUR CSA**: The bank sees a different (often higher) PV because EUR funding is cheaper
>
> This creates "CSA basis"—two parties with different collateral arrangements will value the same trade differently. Disputes arise when one party expects USD CSA but the other operates under EUR CSA.
>
> **Best practice**: Always verify CSA terms before trading. The curve you discount with depends on the collateral you'll post.

---

## 21.7 Risk Decomposition: FX Delta, Rates PV01, and Basis Exposure

Cross-currency instruments have three distinct risk dimensions that must be managed separately. Understanding this decomposition is essential for hedging.

### 21.7.1 FX Delta

The sensitivity to spot FX changes. For an FX forward with PV = $X(0) P_f - K P_d$:

$$\frac{\partial V}{\partial X(0)} = P_f(0,T)$$

This measures how much PV changes (in domestic currency) for a unit change in the spot rate.

### 21.7.2 Interest Rate PV01 / DV01 by Currency

Tuckman defines DV01 as "the change in price for a one-basis-point decline in rates":

$$\text{DV01} = -\frac{\partial P}{\partial y} \times 0.0001$$

For cross-currency portfolios, you compute:
- **Domestic PV01**: Sensitivity to domestic curve shifts
- **Foreign PV01**: Sensitivity to foreign curve shifts, converted to domestic at spot

These are not fungible—you cannot hedge domestic rate risk with foreign instruments (without taking on FX risk).

### 21.7.3 Basis Exposure

The sensitivity to basis spread changes. For a position with basis $e$ appearing in its valuation:

$$\frac{\partial V}{\partial e} = \text{Basis01}$$

This risk is distinct from both FX delta and rates PV01. A portfolio can be FX-hedged and DV01-neutral but still have significant basis exposure.

> **Example 21.7: Basis Shock Sensitivity**
>
> From Example 21.3, the swap PV per unit notional is:
>
> $V = N_d[1 - (B + eA)]$
>
> where $B$ is the "no-basis" floating PV and $A$ is the annuity.
>
> Sensitivity to basis:
>
> $\frac{\partial V}{\partial e} = -N_d A$
>
> For $N_d = \$1{,}000{,}000$ and $A = 0.9925$:
>
> A +5bp basis widening ($\Delta e = +0.0005$):
>
> $\Delta V \approx -1{,}000{,}000 \times 0.9925 \times 0.0005 = -\$496$
>
> This is basis risk—separate from FX and rate moves.

### 21.7.4 Summary of Risk Decomposition

| Risk Type | What It Measures | Hedge Instruments |
|-----------|-----------------|-------------------|
| FX delta | Spot rate sensitivity | FX forwards, spot |
| Domestic PV01 | Domestic curve sensitivity | Domestic swaps, futures |
| Foreign PV01 | Foreign curve sensitivity | Foreign swaps, futures |
| Basis01 | Basis spread sensitivity | Basis swaps |

### 21.7.5 P&L Attribution for Cross-Currency Portfolios

> **Practitioner Note:** P&L attribution (or "P&L explain") is how traders and risk managers decompose daily portfolio changes into their constituent risk factors. This is essential for understanding what drove performance.

**The attribution formula:**

For a cross-currency position, daily P&L can be decomposed as:

$$\boxed{\Delta PV \approx \underbrace{\Delta_{FX} \cdot \delta S}_{\text{FX}} + \underbrace{PV01_d \cdot \delta r_d}_{\text{Dom Rates}} + \underbrace{PV01_f \cdot \delta r_f \cdot S}_{\text{For Rates}} + \underbrace{Basis01 \cdot \delta e}_{\text{Basis}} + \underbrace{\theta}_{\text{Carry}} + \underbrace{\varepsilon}_{\text{Unexplained}}}$$

where:
- $\Delta_{FX}$: FX delta
- $\delta S$: Change in spot FX rate
- $PV01_d$, $PV01_f$: Domestic and foreign rate sensitivities
- $\delta r_d$, $\delta r_f$: Changes in domestic and foreign rates (in bp)
- $Basis01$: Basis spread sensitivity
- $\delta e$: Change in basis spread (in bp)
- $\theta$: Time decay / carry
- $\varepsilon$: Unexplained (higher-order effects, cross-gamma)

> **Example 21.8: Daily P&L Attribution for a 5Y EUR/USD Xccy Swap**
>
> **Position:** Long €100mm notional, receive EUR EURIBOR + 0bp, pay USD SOFR
>
> **Risk exposures:**
> - FX Delta: $-€4.5mm$ (short EUR via the principal exchange)
> - USD PV01: $+\$45,000$ per bp (benefit from lower USD rates)
> - EUR PV01: $-€40,000$ per bp (hurt by lower EUR rates)
> - Basis01: $+\$4,500$ per bp (benefit from basis tightening)
>
> **Day's market moves:**
> - EUR/USD: 1.1000 → 1.1050 (+0.45%)
> - USD 5Y: 4.00% → 4.05% (+5bp)
> - EUR 5Y: 3.00% → 2.95% (-5bp)
> - EUR/USD 5Y basis: -30bp → -25bp (+5bp tighter)
>
> **P&L attribution:**
> | Component | Calculation | P&L |
> |-----------|-------------|-----|
> | FX | $-€4.5mm \times 0.0045$ | $-\$20,250$ |
> | USD Rates | $+\$45,000 \times 5$ | $-\$225,000$ |
> | EUR Rates | $-€40,000 \times (-5) \times 1.1025$ | $+\$220,500$ |
> | Basis | $+\$4,500 \times 5$ | $+\$22,500$ |
> | **Total Explained** | | $-\$2,250$ |
>
> **Interpretation:** The position was approximately rate-neutral (USD and EUR P&L offset), lost slightly on FX, but gained on basis tightening. Net small loss.

> **Desk Reality: When P&L Explain Fails**
>
> In crisis periods, "unexplained" P&L can dominate. This happens because:
> 1. **Cross-gamma**: FX and rates move together in large moves; the linear approximation breaks down
> 2. **Correlation breakdown**: Historical correlations (EUR rates vs EUR/USD spot) shift
> 3. **Liquidity effects**: Bid-ask spreads widen; marks become stale
> 4. **Basis volatility**: Basis moves much more than the linear sensitivity suggests (convexity in basis)
>
> During March 2020, many xccy books had 50%+ unexplained P&L on peak days. This is a warning sign that your risk model is missing something—usually basis convexity or correlation regime shifts.

---

## 21.8 Additional Worked Examples

### Example 21.9: CIP Forward from Zero Rates (Continuous Compounding)

**Given:**
- Domestic = USD, Foreign = EUR
- Spot $S(0) = 1.1000$ USD/EUR
- $r_d = 3.00\%$, $r_f = 1.00\%$ (continuous)
- $T = 0.5$ years

**Solution:**

$$F(0,T) = S(0) \, e^{(r_d - r_f)T} = 1.1000 \times e^{0.02 \times 0.5} = 1.1000 \times 1.010050 = 1.111055 \text{ USD/EUR}$$

**Forward points:** $F - S = 1.111055 - 1.1000 = 0.011055$ USD/EUR

**Interpretation:** Higher domestic rates mean the domestic currency depreciates in the forward (forward > spot in D/F terms).

### Example 21.10: Reconciling Rate and DF Forms

**Using the same data, verify via discount factors:**

$$P_d(0,0.5) = e^{-0.03 \times 0.5} = 0.985112$$
$$P_f(0,0.5) = e^{-0.01 \times 0.5} = 0.995012$$

$$F(0,0.5) = S(0) \times \frac{P_f}{P_d} = 1.1000 \times \frac{0.995012}{0.985112} = 1.1000 \times 1.010050 = 1.111055$$

**Match confirmed.** ✓

### Example 21.11: Arbitrage-Consistency Check

**Scenario:** Compare FX-implied EUR curve against independently-built EUR OIS curve.

**Given:**
- USD discount: $P_d(0,2) = 0.941765$
- Spot: $S(0) = 1.1000$
- Market FX forward: $F_{mkt}(0,2) = 1.1500$ (hypothetical "rich" forward)
- Independent EUR OIS: $P_f^{OIS}(0,2) = 0.980199$ (implying 1% EUR rate)

**FX-implied EUR DF:**
$$P_f^{FX}(0,2) = \frac{F_{mkt}}{S(0)} P_d(0,2) = \frac{1.1500}{1.1000} \times 0.941765 = 1.04545 \times 0.941765 = 0.984572$$

**Discrepancy:**
$$\Delta P = P_f^{FX} - P_f^{OIS} = 0.984572 - 0.980199 = 0.004373$$

**In yield terms:**
- FX-implied EUR rate: $-\frac{1}{2}\ln(0.984572) = 0.777\%$
- OIS EUR rate: $-\frac{1}{2}\ln(0.980199) = 1.000\%$

**Mismatch:** ~22 bp in 2Y rates.

**Interpretation:** The curves are inconsistent. Either:
1. The FX forward is mispriced, or
2. There is a cross-currency basis of ~22 bp that the OIS curve doesn't capture

A consistent multi-curve framework would calibrate to *both* the OIS market and the FX forward market, with basis swaps providing the reconciliation.

### Example 21.12: Hedged Foreign Bond Valuation

**Given:**
- Domestic = USD, Foreign = JPY
- Spot $X(0) = 0.0091$ USD/JPY
- JPY ZCB: ¥100,000,000 payoff at $T = 1$
- Discount factors: $P_f(0,1) = 0.9900$, $P_d(0,1) = 0.970446$

**Unhedged USD PV:**
$$PV_{unhedged} = X(0) \times N_f \times P_f(0,1) = 0.0091 \times 100{,}000{,}000 \times 0.9900 = \$900{,}900$$

**CIP-implied forward:**
$$K^* = X(0) \times \frac{P_f}{P_d} = 0.0091 \times \frac{0.9900}{0.970446} = 0.009284 \text{ USD/JPY}$$

**Hedged PV (sell JPY forward at $K^*$):**
$$PV_{hedged} = K^* \times N_f \times P_d(0,1) = 0.009284 \times 100{,}000{,}000 \times 0.970446 = \$900{,}900$$

**Result:** Under CIP, hedged PV = unhedged PV. ✓

**If basis causes deviation:** Suppose the traded forward is $K_{mkt} = 0.0093$:
$$PV_{hedged,mkt} = 0.0093 \times 100{,}000{,}000 \times 0.970446 = \$902{,}514$$

**Basis gain:** $\$902{,}514 - \$900{,}900 = \$1{,}614$

This "extra" PV is the manifestation of cross-currency basis in hedged foreign asset valuation.

---

## 21.9 Practical Notes

### 21.9.1 Quote Direction Conventions

Hull notes that "in many major pairs the spot/forward exchange rate is normally quoted as the number of units of the currency that are equivalent to one U.S. dollar." This varies by pair:

- EUR/USD: EUR per USD (European terms)
- USD/JPY: JPY per USD (American terms in Japanese quoting)
- GBP/USD: USD per GBP (American terms)

**Impact on formulas:** If your market quote uses the inverse direction, you must invert spot and forward before applying CIP formulas. A common source of errors is misapplying the formula with an inverted quote.

### 21.9.2 Basis Swap Quoting Conventions

**I'm not sure** about universal conventions for which leg receives the spread. The spread can be quoted on:
- The non-USD leg (common)
- The "lower-rate" currency leg
- Different conventions by dealer/venue

**Always confirm** with the specific market before trading or calibrating.

### 21.9.3 Settlement and Calendar Issues

**I'm not sure** about exact spot/settlement conventions, which are currency-pair specific:
- Spot date (T+2 vs T+1)
- Holiday calendars (both currencies' holidays matter)
- Business day conventions for adjusting flows

These affect accrual calculations and must be specified precisely for production systems.

### 21.9.4 Common Implementation Errors

> **Gotcha Box: The Million-Dollar Sign Error**
>
> The most common xccy curve construction error is getting the sign or direction wrong on:
> 1. **FX quote direction**: Is the quote domestic/foreign or foreign/domestic?
> 2. **Basis spread sign**: Does negative basis mean EUR pays or receives?
> 3. **Notional exchange direction**: Who receives which currency at inception?
>
> A sign error on a $100mm 10Y xccy swap with 50bp basis can produce a PV error of:
> $$Error \approx 0.0050 \times 10 \times \$100mm \times 2 = \$10mm$$
>
> **Prevention**: Always sanity-check that higher interest rate currencies should trade at forward discounts, and that USD generally trades with a premium (non-USD legs typically pay negative spreads).

**Common errors and how to avoid them:**

| Error | Symptom | Prevention |
|-------|---------|------------|
| Inverted FX quote | Forward points have wrong sign | Verify: higher rates → forward discount |
| Wrong basis leg | PV way off market | Check market convention for pair |
| Day count mismatch | Small but persistent errors | Verify both legs use correct convention |
| Settlement date misalignment | PV breaks on date transitions | Align to correct spot/fixing dates |
| Using wrong discount curve | PV differs from Street marks | Match collateral currency to CSA |

**Sign convention cheat sheet:**

| Currency Pair | Typical Basis | Quote Convention |
|---------------|---------------|------------------|
| EUR/USD | -15 to -50bp | Spread on EUR leg |
| USD/JPY | -20 to -80bp | Spread on JPY leg |
| GBP/USD | -10 to -30bp | Spread on GBP leg |
| USD/CHF | -20 to -60bp | Spread on CHF leg |

**Note**: These ranges are indicative of post-2015 typical levels; actual values vary with market conditions.

---

## Summary

1. **CIP is the foundation**: The no-arbitrage relationship $F = S \cdot P_f/P_d$ links spot FX, forward FX, and discount factors across currencies. Hull's rate form $F_0 = S_0 e^{(r_d - r_f)T}$ is equivalent under continuous compounding.

2. **FX forwards constrain curves**: Given a domestic curve and FX forward quotes, the foreign curve is determined by CIP. Building curves independently creates arbitrage.

3. **Cross-currency basis is persistent**: Post-2008, the basis between FX-implied curves and local curves is material (often 10-50+ bp in stress) due to funding segmentation, balance sheet constraints, and credit differentiation. The basis represents the "price of USD liquidity"—the cost for non-US banks to access dollar funding.

4. **Basis swaps extend the constraint horizon**: Beyond ~1 year, cross-currency basis swaps replace FX forwards as the source of constraint information. Andersen and Piterbarg note that "the interbank FX forward market is rarely liquid beyond maturities of one year."

5. **Multi-curve construction is iterative**: Build domestic curves first, then calibrate foreign discount curves to basis swaps using the Andersen-Piterbarg algorithm. The algorithm converges quickly because index ≈ discount is a good initial approximation.

6. **Collateral currency matters**: The currency in which collateral is posted determines the appropriate discount curve. A USD swap with EUR collateral values differently than one with USD collateral.

7. **Risk decomposes into four dimensions**: FX delta, domestic rates PV01, foreign rates PV01, and basis exposure are distinct and require separate hedges. P&L attribution decomposes daily changes into these components.

8. **Market dynamics drive basis**: Corporate issuance arbitrage (issuing in low-rate currencies and swapping), bank ALM demand for USD, and quarter-end regulatory effects all influence basis levels and direction.

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **Covered Interest Parity (CIP)** | $F = S \cdot P_f/P_d$ | Foundation for cross-currency no-arbitrage |
| **FX-implied curve** | Foreign DFs derived from domestic curve + FX forwards | Ensures curve consistency across currencies |
| **Cross-currency basis** | Yield spread between index and discount curves: $P^{(L)} = P e^{-st}$ | Captures funding/credit segmentation; the "price of USD liquidity" |
| **Basis swap** | Floating-floating xccy swap with spread | Provides constraint info for long maturities |
| **Bedrock assumption** | USD index = USD discount (or use OIS) | Anchors the multi-curve system |
| **Forward FX rate** | $X_T(t) = X(t) P_f(t,T)/P_d(t,T)$ | The FX rate locked in today for future delivery |
| **USD premium** | Non-US banks pay to swap into USD | Explains persistent negative basis on non-USD legs |
| **Issuance arbitrage** | Issue in low-rate currency, swap to funding currency | Corporate exploitation of basis; drives basis dynamics |
| **Collateral-curve linkage** | Discount at the rate earned on collateral | CSA currency determines appropriate discount curve |
| **P&L attribution** | Decompose PV changes by risk factor | Essential for understanding what drove daily performance |

---

## Notation for This Chapter

| Symbol | Definition |
|--------|------------|
| $d$, $f$ | Domestic and foreign currencies |
| $X(0)$ or $S_0$ | Spot FX (domestic per foreign) |
| $F(0,T)$ or $X_T(0)$ | Forward FX for maturity $T$ |
| $P_d(0,T)$, $P_f(0,T)$ | Discount factors |
| $P^{(L)}(0,T)$ | Index/projection curve |
| $s(t)$ | CRX yield spread |
| $e$ | Basis swap spread |
| $\tau$ | Year fraction / accrual |
| $r_d$, $r_f$ | Domestic and foreign risk-free rates |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What does CIP relate? | Spot FX, forward FX, and discount factors across two currencies |
| 2 | CIP in discount-factor form? | $F(0,T) = X(0) \cdot P_f(0,T)/P_d(0,T)$ |
| 3 | CIP in Hull's rate form? | $F_0 = S_0 e^{(r_d - r_f)T}$ |
| 4 | How do you derive $P_f$ from $F$, $X$, and $P_d$? | $P_f = (F/X) \cdot P_d$ |
| 5 | What is the cross-currency yield spread formula? | $P^{(L)}(t) = P(t) e^{-s(t)t}$, so $s(t) = -\frac{1}{t}\ln(P^{(L)}/P)$ |
| 6 | Why can independent curve construction create arbitrage? | Curves may violate the CIP constraint (6.38), allowing costless round-trip profit |
| 7 | What is a cross-currency basis swap? | Swap exchanging floating rates in two currencies + spread, with notional exchanges at spot ratio |
| 8 | Why are basis swaps needed for long maturities? | FX forwards are "rarely liquid beyond maturities of one year" (Andersen) |
| 9 | What was the JPY CRX basis in late 1990s? | Around -40 bp due to Japanese bank credit concerns |
| 10 | What was the JPY CRX basis in early 2008? | Up to +60 bp due to hedging demand from FX book rebalancing |
| 11 | What is the "bedrock" USD assumption? | $P_\$^{(L)} = P_\$$ (USD index = USD discount) |
| 12 | Why doesn't arbitrage eliminate the basis? | Capital requirements, counterparty limits, term funding constraints, balance sheet costs |
| 13 | What three risks does a xccy position have? | FX delta, rates PV01 (by currency), basis exposure |
| 14 | What is basis01? | PV sensitivity to a 1bp change in basis spread |
| 15 | How is the iterative curve construction initiated? | Set $V^{(L)}(0) = V$ (prices as if index = discount) |
| 16 | Why avoid using xccy instruments to infer USD basis? | Creates circular reasoning; use domestic OIS/basis swaps instead |
| 17 | What constrains the short end of xccy curves? | FX forwards (liquid to ~1 year) |
| 18 | What constrains the long end of xccy curves? | Cross-currency basis swaps (quoted to 30+ years) |
| 19 | One-period basis swap = ? | FX forward contract |
| 20 | If $P_d = P_f$, what is $F$? | $F = X$ (forward equals spot when rates are equal) |
| 21 | What does negative EUR/USD basis mean? | EUR-funded party pays to swap into USD (USD is more expensive) |
| 22 | Why does basis widen at quarter-end? | Regulatory window-dressing compresses bank balance sheets, reducing xccy supply |
| 23 | What discount curve for EUR-collateralized USD swap? | USD OIS adjusted for basis (not pure USD OIS) |
| 24 | What are Fed swap lines? | Central bank arrangements allowing foreign CBs to borrow USD from Fed, acting as basis ceiling |
| 25 | What happened to EUR/USD basis in March 2020? | Widened from -20bp to -150bp, then compressed after Fed swap line expansion |
| 26 | Why do corporates issue in EUR and swap to USD? | Exploit tighter EUR credit spreads and receive basis; can achieve lower all-in USD funding |
| 27 | What is the P&L attribution formula for xccy? | $\Delta PV = \Delta_{FX} \cdot \delta S + PV01_d \cdot \delta r_d + PV01_f \cdot \delta r_f + Basis01 \cdot \delta e + \theta$ |

---

## Mini Problem Set

### Problem 1 (Basic)
Given $S(0) = 1.25$ USD/EUR, $r_d = 4\%$, $r_f = 1.5\%$, $T = 1$ (continuous), compute $F(0,1)$.

**Solution:** $F = 1.25 \times e^{0.025} = 1.2817$

### Problem 2 (Basic)
Given $X(0) = 1.10$, $P_d(0,2) = 0.94$, $P_f(0,2) = 0.98$, compute $F(0,2)$.

**Solution:** $F = 1.10 \times 0.98/0.94 = 1.1468$

### Problem 3 (Basic)
Derive $P_f(0,T)$ from $X(0)$, $F(0,T)$, and $P_d(0,T)$.

**Solution:** Rearrange CIP: $P_f = (F/X) \cdot P_d$

### Problem 4 (Intermediate)
An FX forward receives 1 EUR at $T$ and pays $K$ USD. Write the PV at time 0 and solve for $K$ that makes PV = 0.

**Solution:** $PV = X(0) P_f - K P_d$. Set to zero: $K = X(0) P_f/P_d = F$

### Problem 5 (Intermediate)
Using $P^{(L)}(t) = P(t) e^{-s(t)t}$, solve for $s(t)$ given $P^{(L)}(1) = 0.9885$ and $P(1) = 0.9900$.

**Solution:** $s(1) = -\ln(0.9885/0.9900) = -\ln(0.99848) = 0.00152 = 15.2$ bp

### Problem 6 (Intermediate)
Explain qualitatively why building USD and EUR curves independently can create arbitrage.

**Solution:** If the curves don't satisfy $P_{EUR} = (F/X) \cdot P_{USD}$, one can convert USD → EUR → USD via three costless swaps (USD IRS, xccy basis, EUR IRS) and end up with a different value than direct USD discounting—a riskless profit.

### Problem 7 (Advanced)
Build a 3-point FX-implied EUR curve from:
- USD curve: $P_d(0.5) = 0.99$, $P_d(1) = 0.97$, $P_d(2) = 0.93$
- Spot: $X = 1.15$
- Forwards: $F(0.5) = 1.16$, $F(1) = 1.18$, $F(2) = 1.22$

Verify discount factors are monotonically decreasing.

**Solution:**
- $P_f(0.5) = (1.16/1.15) \times 0.99 = 0.9986$
- $P_f(1) = (1.18/1.15) \times 0.97 = 0.9956$
- $P_f(2) = (1.22/1.15) \times 0.93 = 0.9866$

Check: $0.9986 > 0.9956 > 0.9866$ ✓

### Problem 8 (Advanced)
In a two-curve setup, show that if projection and discount curves coincide ($P^{(L)} = P$), the PV of a floating leg with principal repayment telescopes to par.

**Solution:** Floating leg PV:
$$\sum_{i=0}^{n-1} L_i \tau_i P(t_{i+1}) + P(t_n)$$

With $L_i = \frac{1}{\tau_i}\left(\frac{P(t_i)}{P(t_{i+1})} - 1\right)$:
$$\sum_{i=0}^{n-1} \left(\frac{P(t_i)}{P(t_{i+1})} - 1\right) P(t_{i+1}) + P(t_n) = \sum_{i=0}^{n-1}(P(t_i) - P(t_{i+1})) + P(t_n)$$

This telescopes: $(P(0) - P(t_1)) + (P(t_1) - P(t_2)) + \cdots + P(t_n) = P(0) = 1$ ✓

### Problem 9 (Advanced)
Using Hull's AUD/USD example, if 2-year rates are 3% (AUD) and 1% (USD), spot is 0.75 USD/AUD, and the 2-year forward trades at 0.7600 (vs fair value 0.7206), describe the arbitrage strategy.

**Solution:**
1. Borrow 1,000 USD at 1% for 2 years → owe $1000 e^{0.02} = 1020.20$ at maturity
2. Convert to $1000/0.75 = 1333.33$ AUD, invest at 3% → receive $1333.33 e^{0.06} = 1415.79$ AUD
3. Sell 1415.79 AUD forward at 0.76 → receive $1415.79 \times 0.76 = 1075.99$ USD
4. Repay USD loan ($1020.20), keep profit of $55.79

### Problem 10 (Intermediate — P&L Attribution)
A 5Y EUR/USD xccy swap has: FX delta = -€5mm, USD PV01 = +$50k/bp, EUR PV01 = -€45k/bp, Basis01 = +$5k/bp.

Over one day: EUR/USD moves from 1.10 to 1.12, USD rates +3bp, EUR rates -2bp, basis +2bp tighter.

Attribute the P&L.

**Solution:**
- FX: $-€5mm \times 0.02 / 1.10 = -\$90,909$
- USD: $+\$50k \times 3 = -\$150,000$ (rates up hurts if receiving fixed)
- EUR: $-€45k \times (-2) \times 1.11 = +\$99,900$
- Basis: $+\$5k \times 2 = +\$10,000$
- **Total**: $-\$131,009$

### Problem 11 (Intermediate — Issuance Arbitrage)
A corporate can issue:
- USD bonds at T+90bp = 4.90%
- EUR bonds at Bunds+70bp = 3.70%

EUR 5Y swap rate is 3.00%, USD 5Y swap rate is 4.00%, basis is -25bp on EUR leg.

Calculate the all-in USD funding cost via EUR issuance + swap.

**Solution:**
- EUR spread: 70bp over swaps
- Swap EUR for USD: pay USD SOFR, receive EUR EURIBOR
- Basis: receive -25bp on EUR (i.e., pay EUR EURIBOR - 25bp)
- Net: USD swap rate + EUR spread - basis benefit = 4.00% + 0.70% - 0.25% = 4.45%
- vs. direct USD: 4.90%
- **Savings: 45bp**

### Problem 12 (Advanced — Collateral Effect)
A 5Y USD IRS (pay fixed 4.00%, receive SOFR) has PV01 of $48,000/bp. USD OIS is 4.20%. If the swap is collateralized in EUR instead of USD, and EUR/USD basis is -30bp, estimate the PV difference between EUR-collateral and USD-collateral.

**Solution:**
- EUR collateral effectively discounts at ~30bp lower rate (basis benefit)
- Duration ≈ 4.8 years (from PV01/$100mm)
- PV impact ≈ $\frac{0.0030 \times 4.8 \times \$100mm}{1} = \$1.44mm$
- The swap is worth **more** under EUR collateral (lower discounting rate increases receiver leg PV)

*Note: This is a simplified approximation; full calculation requires proper multi-curve valuation.*

---

## Source Map

### (A) Book-Verified Facts

| Fact | Source |
|------|--------|
| CIP rate form: $F_0 = S_0 e^{(r_d - r_f)T}$ | Hull Ch 5, equation (5.9) |
| "Interest rate parity relationship from international finance" | Hull Ch 5 |
| CIP discount-factor form: $F(0,T) = X(0) P_f/P_d$ | Andersen Vol 1 Ch 6 (implicit in eq 6.38) |
| Forward FX rate definition: $X_T(t) = X(t) P_f(t,T)/P_d(t,T)$ | Andersen Vol 1 Ch 4.3.1 |
| FX forward replication via foreign/domestic ZCBs | Andersen Vol 1 Ch 4.3.1 |
| Independent curve construction violates (6.38), creating arbitrage | Andersen Vol 1 Ch 6.5.2.1 |
| Multi-curve separation: discount vs index curves | Andersen Vol 1 Ch 6.5.2.2 |
| Cross-currency basis swaps: exchange floating + spread, notional at spot ratio | Andersen Vol 1 Ch 6.5.2.3 |
| One-period CRX basis swap = FX forward | Andersen Vol 1 Ch 6.5.2.3 |
| CRX yield spread definition: $P^{(L)}(t) = P(t) e^{-s(t)t}$ | Andersen Vol 1 Ch 6.5.2.2 |
| JPY basis -40 bp in late 1990s | Andersen Vol 1 Ch 6.5.2.2 |
| JPY basis +60 bp in early 2008 | Andersen Vol 1 Ch 6.5.2.2 |
| FX forwards rarely liquid beyond ~1 year | Andersen Vol 1 Ch 6.5.2.3 |
| DV01 definition | Tuckman Ch 5-6 |
| FX quote direction warning | Hull Ch 5 |
| Basis swap valuation equation (6.42) | Andersen Vol 1 Ch 6.5.2.3 |
| Iterative algorithm for cross-currency curve construction | Andersen Vol 1 Ch 6.5.2.4 |
| "Bedrock" assumption (Assumption 6.5.1) | Andersen Vol 1 Ch 6.5.2.2 |
| AUD/USD arbitrage example | Hull Ch 5, Example 5.6 |
| Fed funds/Libor spread widening post-2007 | Andersen Vol 1 Ch 6.5.3 |
| "Uncollateralized derivatives subject to credit risk" | Andersen Vol 1 Ch 6.5.3 |
| OIS and Fed funds/Libor basis swap markets for USD curve | Andersen Vol 1 Ch 6.5.3 |
| Iteration converges quickly | Andersen Vol 1 Ch 6.5.2.4 |

### (B) Claude-Extended Content (Practitioner Notes)

| Content | Context |
|---------|---------|
| Balance sheet economics of "arbitrage" | Extended from general knowledge of Basel III constraints |
| The USD Premium section (21.3.5) | Structural market dynamics, not in standard textbooks |
| Trading the Basis section (21.3.6) | Corporate issuance arbitrage, Yankee funding dynamics |
| Quarter-end basis widening | Regulatory window-dressing effects |
| March 2020 COVID basis blow-out | Recent historical episode (post-textbook publication) |
| Central bank swap lines as circuit breakers | Post-2008 policy innovation |
| Mark-to-market basis swap structures | Modern market practice |
| Collateral currency section (21.6) | Operational practice extending textbook theory |
| P&L attribution framework (21.7.5) | Desk-level practice for risk management |
| Sign convention cheat sheet | Market convention reference |

### (C) Reasoned Inference — Derivation Logic

| Inference | Derivation |
|-----------|------------|
| Implied foreign DF: $P_f = (F/X) P_d$ | Algebra from CIP-DF form |
| Yield spread formula: $s(t) = -(1/t)\ln(P^{(L)}/P)$ | Algebra from spread definition |
| Floating leg telescopes to par when $P^{(L)} = P$ | Forward rate substitution and telescoping sum |
| Basis sensitivity formula | Differentiation of swap PV w.r.t. spread |
| Forward equals spot when rates equal | Setting $P_d = P_f$ in CIP formula |
| P&L attribution formula | Linear approximation from Taylor expansion of PV function |
| Numerical iteration example (21.5.3) | Application of Andersen algorithm to specific numbers |

### (D) Flagged Uncertainties

| Topic | Uncertainty |
|-------|-------------|
| Spot/settlement conventions | Currency-pair specific; not fully specified in sources |
| Basis swap spread conventions (which leg, sign) | Market-convention specific; varies by venue |
| Calendar construction | Books do not provide full operational calendars |
| MTM basis swap exact mechanics | Varies by currency pair and counterparty |
| Multi-currency CSA optionality pricing | Requires basis volatility assumptions not covered |
| Typical basis ranges by currency | Indicative only; varies significantly with market conditions |

---

*Last Updated: January 26, 2026*
