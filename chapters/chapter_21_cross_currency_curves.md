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
3. **Cross-Currency Basis** (Section 21.3): What "basis" means at the curve level, why it exists, and how to measure it.
4. **Cross-Currency Basis Swaps** (Section 21.4): The instruments that provide constraint information beyond the short-dated FX forward market.
5. **Multi-Currency Curve Construction** (Section 21.5): A practical algorithm for building consistent cross-currency curves.
6. **Risk Decomposition** (Section 21.6): FX delta, rates PV01, and basis exposure as distinct risk dimensions.

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

### 21.5.3 Avoiding Circularity

A subtle issue arises: if we use cross-currency basis swaps to infer the discounting basis in USD, we create circular reasoning—we're using non-USD markets to determine USD discounting.

Andersen and Piterbarg's resolution: "To estimate this basis in USD, we need to rely on domestic markets only; doing otherwise will introduce a circularity into our arguments."

In practice, the USD Libor-OIS basis is calibrated from:
- OIS swaps (overnight index swaps)
- Fed Funds / LIBOR basis swaps

Then cross-currency instruments translate this basis into other currencies. The key insight is that the USD curve must be self-consistent before extending to other currencies.

---

## 21.6 Risk Decomposition: FX Delta, Rates PV01, and Basis Exposure

Cross-currency instruments have three distinct risk dimensions that must be managed separately. Understanding this decomposition is essential for hedging.

### 21.6.1 FX Delta

The sensitivity to spot FX changes. For an FX forward with PV = $X(0) P_f - K P_d$:

$$\frac{\partial V}{\partial X(0)} = P_f(0,T)$$

This measures how much PV changes (in domestic currency) for a unit change in the spot rate.

### 21.6.2 Interest Rate PV01 / DV01 by Currency

Tuckman defines DV01 as "the change in price for a one-basis-point decline in rates":

$$\text{DV01} = -\frac{\partial P}{\partial y} \times 0.0001$$

For cross-currency portfolios, you compute:
- **Domestic PV01**: Sensitivity to domestic curve shifts
- **Foreign PV01**: Sensitivity to foreign curve shifts, converted to domestic at spot

These are not fungible—you cannot hedge domestic rate risk with foreign instruments (without taking on FX risk).

### 21.6.3 Basis Exposure

The sensitivity to basis spread changes. For a position with basis $e$ appearing in its valuation:

$$\frac{\partial V}{\partial e} = \text{Basis01}$$

This risk is distinct from both FX delta and rates PV01. A portfolio can be FX-hedged and DV01-neutral but still have significant basis exposure.

> **Example 21.4: Basis Shock Sensitivity**
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

### 21.6.4 Summary of Risk Decomposition

| Risk Type | What It Measures | Hedge Instruments |
|-----------|-----------------|-------------------|
| FX delta | Spot rate sensitivity | FX forwards, spot |
| Domestic PV01 | Domestic curve sensitivity | Domestic swaps, futures |
| Foreign PV01 | Foreign curve sensitivity | Foreign swaps, futures |
| Basis01 | Basis spread sensitivity | Basis swaps |

---

## 21.7 Additional Worked Examples

### Example 21.5: CIP Forward from Zero Rates (Continuous Compounding)

**Given:**
- Domestic = USD, Foreign = EUR
- Spot $S(0) = 1.1000$ USD/EUR
- $r_d = 3.00\%$, $r_f = 1.00\%$ (continuous)
- $T = 0.5$ years

**Solution:**

$$F(0,T) = S(0) \, e^{(r_d - r_f)T} = 1.1000 \times e^{0.02 \times 0.5} = 1.1000 \times 1.010050 = 1.111055 \text{ USD/EUR}$$

**Forward points:** $F - S = 1.111055 - 1.1000 = 0.011055$ USD/EUR

**Interpretation:** Higher domestic rates mean the domestic currency depreciates in the forward (forward > spot in D/F terms).

### Example 21.6: Reconciling Rate and DF Forms

**Using the same data, verify via discount factors:**

$$P_d(0,0.5) = e^{-0.03 \times 0.5} = 0.985112$$
$$P_f(0,0.5) = e^{-0.01 \times 0.5} = 0.995012$$

$$F(0,0.5) = S(0) \times \frac{P_f}{P_d} = 1.1000 \times \frac{0.995012}{0.985112} = 1.1000 \times 1.010050 = 1.111055$$

**Match confirmed.** ✓

### Example 21.7: Arbitrage-Consistency Check

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

### Example 21.8: Hedged Foreign Bond Valuation

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

## 21.8 Practical Notes

### 21.8.1 Quote Direction Conventions

Hull notes that "in many major pairs the spot/forward exchange rate is normally quoted as the number of units of the currency that are equivalent to one U.S. dollar." This varies by pair:

- EUR/USD: EUR per USD (European terms)
- USD/JPY: JPY per USD (American terms in Japanese quoting)
- GBP/USD: USD per GBP (American terms)

**Impact on formulas:** If your market quote uses the inverse direction, you must invert spot and forward before applying CIP formulas. A common source of errors is misapplying the formula with an inverted quote.

### 21.8.2 Basis Swap Quoting Conventions

**I'm not sure** about universal conventions for which leg receives the spread. The spread can be quoted on:
- The non-USD leg (common)
- The "lower-rate" currency leg
- Different conventions by dealer/venue

**Always confirm** with the specific market before trading or calibrating.

### 21.8.3 Settlement and Calendar Issues

**I'm not sure** about exact spot/settlement conventions, which are currency-pair specific:
- Spot date (T+2 vs T+1)
- Holiday calendars (both currencies' holidays matter)
- Business day conventions for adjusting flows

These affect accrual calculations and must be specified precisely for production systems.

### 21.8.4 When Collateral Currency Differs

The discussion in this chapter assumes standard collateral practices. Andersen and Piterbarg note that "uncollateralized derivative contracts are subject to credit risk, and a fully consistent pricing approach needs to incorporate the cost of hedging this risk (the so-called credit valuation adjustment or CVA)."

When collateral is posted in a third currency, additional basis adjustments may be needed. This is outside the scope of this chapter but represents an important practical consideration for exotic cross-currency trades.

---

## Summary

1. **CIP is the foundation**: The no-arbitrage relationship $F = S \cdot P_f/P_d$ links spot FX, forward FX, and discount factors across currencies. Hull's rate form $F_0 = S_0 e^{(r_d - r_f)T}$ is equivalent under continuous compounding.

2. **FX forwards constrain curves**: Given a domestic curve and FX forward quotes, the foreign curve is determined by CIP. Building curves independently creates arbitrage.

3. **Cross-currency basis is persistent**: Post-2008, the basis between FX-implied curves and local curves is material (often 10-50+ bp in stress) due to funding segmentation, balance sheet constraints, and credit differentiation.

4. **Basis swaps extend the constraint horizon**: Beyond ~1 year, cross-currency basis swaps replace FX forwards as the source of constraint information. Andersen and Piterbarg note that "the interbank FX forward market is rarely liquid beyond maturities of one year."

5. **Multi-curve construction is iterative**: Build domestic curves first, then calibrate foreign discount curves to basis swaps using the Andersen-Piterbarg algorithm.

6. **Risk decomposes into three dimensions**: FX delta, rates PV01 (by currency), and basis exposure are distinct and require separate hedges.

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **Covered Interest Parity (CIP)** | $F = S \cdot P_f/P_d$ | Foundation for cross-currency no-arbitrage |
| **FX-implied curve** | Foreign DFs derived from domestic curve + FX forwards | Ensures curve consistency across currencies |
| **Cross-currency basis** | Yield spread between index and discount curves: $P^{(L)} = P e^{-st}$ | Captures funding/credit segmentation |
| **Basis swap** | Floating-floating xccy swap with spread | Provides constraint info for long maturities |
| **Bedrock assumption** | USD index = USD discount (or use OIS) | Anchors the multi-curve system |
| **Forward FX rate** | $X_T(t) = X(t) P_f(t,T)/P_d(t,T)$ | The FX rate locked in today for future delivery |

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
| 12 | Why doesn't arbitrage eliminate the basis? | Capital requirements, counterparty limits, term funding constraints |
| 13 | What three risks does a xccy position have? | FX delta, rates PV01 (by currency), basis exposure |
| 14 | What is basis01? | PV sensitivity to a 1bp change in basis spread |
| 15 | How is the iterative curve construction initiated? | Set $V^{(L)}(0) = V$ (prices as if index = discount) |
| 16 | Why avoid using xccy instruments to infer USD basis? | Creates circular reasoning; use domestic OIS/basis swaps instead |
| 17 | What constrains the short end of xccy curves? | FX forwards (liquid to ~1 year) |
| 18 | What constrains the long end of xccy curves? | Cross-currency basis swaps (quoted to 30+ years) |
| 19 | One-period basis swap = ? | FX forward contract |
| 20 | If $P_d = P_f$, what is $F$? | $F = X$ (forward equals spot when rates are equal) |

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

---

## Source Map

### (A) Verified Facts — Cite Specific Sources

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

### (B) Reasoned Inference — Derivation Logic

| Inference | Derivation |
|-----------|------------|
| Implied foreign DF: $P_f = (F/X) P_d$ | Algebra from CIP-DF form |
| Yield spread formula: $s(t) = -(1/t)\ln(P^{(L)}/P)$ | Algebra from spread definition |
| Floating leg telescopes to par when $P^{(L)} = P$ | Forward rate substitution and telescoping sum |
| Basis sensitivity formula | Differentiation of swap PV w.r.t. spread |
| Forward equals spot when rates equal | Setting $P_d = P_f$ in CIP formula |

### (C) Flagged Uncertainties

| Topic | Uncertainty |
|-------|-------------|
| Spot/settlement conventions | Currency-pair specific; not fully specified in sources |
| Basis swap spread conventions (which leg, sign) | Market-convention specific; varies by venue |
| Calendar construction | Books do not provide full operational calendars |
| Collateral currency discounting | Flagged as out of scope; additional adjustments needed for third-currency collateral |

---

*Last Updated: January 2026*
