# Chapter 21: Cross-Currency Curves — CIP, FX Forwards, and Cross-Currency Basis as Curve Constraints

---

## Learning Objectives
- Translate an FX forward or xccy basis quote into a constraint linking discount factors across currencies.
- Build an “FX-implied” foreign discount curve from a domestic curve and a strip of FX forwards, and sanity-check the result.
- Explain what the cross-currency basis swap spread is quoting (which leg, which index) and why quote conventions matter.
- Outline a practical multi-currency curve-building workflow (anchor curve → linked curve(s) → consistency checks).
- Decompose cross-currency PV and P&L into FX delta, per-currency DV01, and Basis01 with explicit units and sign.

## Introduction

Prerequisites: [Chapter 18](chapter_18_ois_discounting_curve.md), [Chapter 19](chapter_19_projection_curves_libor_sofr_multi_curve.md), [Chapter 20](chapter_20_tenor_basis.md) (optional refresher: [Chapter 11](chapter_11_dv01_pv01_definitions_computation.md))  
Follow-on: [Chapter 22](chapter_22_multi_curve_risk_jacobians.md), [Chapter 30](chapter_30_fx_swaps_cross_currency_swaps.md), [Chapter 31](chapter_31_multi_currency_risk.md)

A trader in London wants to fund a USD position using EUR. She could borrow EUR, convert to USD at spot, invest the dollars, and hedge the FX exposure with a forward. In a frictionless setting this “round trip” is pinned down by **covered interest parity (CIP)**: the FX forward must be consistent with the interest rate differential, otherwise a cash-and-carry trade produces an arbitrage.

Cross-currency swap and basis quotes are convention-heavy: which leg carries the spread, which index is referenced, and the sign convention all vary by product, venue, and currency pair. A common legacy framing is “foreign floating + \(e\) vs USD floating flat,” but you should treat the term sheet (or the market-data definition) as the source of truth. The practical point for curve construction is that the basis is a traded input that can be materially non-zero, especially in stressed funding regimes.

For curve construction, the key takeaway is operational: **FX forwards and cross-currency basis swaps impose constraints that your multi-currency curves must satisfy.** If you bootstrap USD and JPY curves independently and ignore the FX side, the resulting curves will generally violate these constraints and imply inconsistencies.

> **Analogy: The Teleporter**
>
> Imagine you want 100 EUR in one year. You have two ways to get it:
> 1.  **The Local Path**: Wait in Europe. (Invest EUR today).
> 2.  **The Teleporter**: Wait in the US, then teleport. (Invest USD today, then swap to EUR).
>
> **Covered Interest Parity (CIP)** says these two paths must cost exactly the same. The FX Forward market is the "Teleporter." If the Teleporter is cheaper than the Local Path, everyone will use it, forcing the price back in line.
>
> **When basis is non-zero**: The Teleporter is still working, but it charges a fee (the basis) observed in the xccy swap/basis market.

This chapter develops the machinery for cross-currency curve construction. We will cover:

1. **Covered Interest Parity** (Section 21.1): The no-arbitrage relationship linking spot FX, forward FX, and discount factors across currencies—the foundation for everything that follows.
2. **FX Forwards as Curve Constraints** (Section 21.2): How forward quotes, combined with a domestic curve, pin down the foreign discount curve.
3. **Cross-Currency Basis** (Section 21.3): What "basis" means at the curve level, why it exists, why it persists despite "arbitrage," and what drives it (including demand–supply and issuance/funding flows).
4. **Cross-Currency Basis Swaps** (Section 21.4): The instruments that provide constraint information beyond the short-dated FX forward market, including structural details and cashflow mechanics.
5. **Multi-Currency Curve Construction** (Section 21.5): A practical algorithm for building consistent cross-currency curves, with a complete numerical walkthrough.
6. **Collateral Currency and Curve Choice** (Section 21.6): How the currency of posted collateral affects which discount curve applies—essential for production systems.
7. **Risk Decomposition** (Section 21.7): FX delta, rates DV01, and basis exposure as distinct risk dimensions, including P&L attribution frameworks.

**Chapter boundaries:** This chapter focuses on the *curve construction implications* of cross-currency markets. Chapter 29 covers FX forward mechanics and pricing in detail; Chapter 30 covers cross-currency swap structures and valuation. Here, we treat these instruments primarily as sources of constraint equations that must be satisfied when building curves. Chapter 20 develops the multi-curve framework for a single currency (tenor basis); this chapter extends that framework to multiple currencies.

---

## 21.1 Covered Interest Parity (CIP): The Foundation

Covered interest parity (CIP) is the no-arbitrage relationship that links spot FX, forward FX, and interest rates (or discount factors) across two currencies. It is *covered* because the FX exposure is hedged with a forward contract, eliminating currency risk from the arbitrage argument.

### 21.1.1 CIP in Rate Form

In a simple continuous-compounding setting, the currency forward identity can be written:

$$\boxed{F_0 = S_0 \, e^{(r_d - r_f)T}}$$

where $S_0$ is the spot exchange rate (domestic currency per unit of foreign), $r_d$ is the domestic risk-free rate, $r_f$ is the foreign risk-free rate, and $T$ is the time to maturity.

**Replication argument.** Consider a firm with 1,000 units of domestic currency wanting exposure to foreign currency at time $T$:

- **Strategy A**: Buy foreign currency spot, invest at the foreign rate, ending with $1000/S_0 \times e^{r_f T}$ units of foreign currency.
- **Strategy B**: Invest domestically at $r_d$, convert at time $T$ using a forward at rate $F_0$, ending with $(1000 \times e^{r_d T})/F_0$ units of foreign currency.

No-arbitrage requires the two strategies to give the same foreign-currency amount at $T$:

$$\frac{1000 \times e^{r_f T}}{S_0} = \frac{1000 \times e^{r_d T}}{F_0}$$

Rearranging yields the CIP formula.

> **The "Free Lunch" Calculation**
>
> **Setup:** 2-year rates are 3% (AUD) and 1% (USD), and spot is 0.7500 USD per AUD.
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

For curve construction, the discount-factor formulation is more useful because it avoids specifying compounding conventions. Consider a forward contract to receive 1 unit of foreign currency at time $T$ in exchange for $K$ units of domestic currency. The fundamental constraint is:

$$\boxed{F(0,T) = X(0) \frac{P_f(0,T)}{P_d(0,T)}}$$

where $X(0)$ is the spot FX rate (domestic per foreign), and $P_d, P_f$ are the domestic and foreign discount factors.

**Derivation.** The time-0 present value of each leg is:

- **Foreign leg (receive)**: The domestic PV of 1 foreign at $T$ is $X(0) P_f(0,T)$
- **Domestic leg (pay)**: The domestic PV of paying $K$ at $T$ is $K P_d(0,T)$

Setting PV = 0 at inception (the definition of a forward rate):

$$X(0) P_f(0,T) - K P_d(0,T) = 0 \quad \Rightarrow \quad K = X(0) \frac{P_f(0,T)}{P_d(0,T)} = F(0,T)$$

Equivalently, at an intermediate time $t$ the forward FX rate for maturity $T$ is:

$$X_T(t) = X(t) \frac{P_f(t,T)}{P_d(t,T)}$$

**Arbitrage intuition.** You can replicate a long forward by buying the foreign zero-coupon bond (so you receive 1 unit of foreign at $T$) and financing it by shorting the domestic zero-coupon bond (so you owe a fixed domestic amount at $T$). Zero initial value pins down the forward rate.

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

From the discount-factor form of CIP,

$$\boxed{F(0,T) = X(0)\frac{P_f(0,T)}{P_d(0,T)}}$$

we can solve for the foreign discount factor:

$$\boxed{P_f(0,T) = \frac{F(0,T)}{X(0)} P_d(0,T)}$$

This is the **FX-implied foreign discount factor** at maturity $T$: given an anchor domestic curve $P_d$ and observed spot/forward FX quotes, $P_f$ is pinned down.

> **Pitfall — FX quote direction:** CIP is algebraically simple, but easy to misapply when spot/forward are quoted in the inverse direction.
> **Why it matters:** Inverting $X$ or $F$ incorrectly can create $P_f(0,T)>1$ or non-monotone discount factors, and “arbitrage” that is just a sign error.
> **Quick check:** In domestic-per-foreign terms, if domestic rates are higher than foreign rates, you should typically see $F(0,T) > X(0)$.

### 21.2.2 What Happens When Curves Are Built Independently

Bootstrapping each currency’s curve from its local swap market and ignoring FX forwards/basis swaps generally produces a curve set that violates the cross-currency constraint above. Even if the violation is “small” in rate terms, it can be large in PV for multi-currency books.

A useful mental model is to compare two ways of turning a domestic fixed cashflow into a foreign fixed cashflow:
1. Convert fixed domestic to domestic floating (domestic IRS).
2. Exchange domestic floating for foreign floating plus a spread (xccy basis swap).
3. Convert foreign floating to foreign fixed (foreign IRS).

If your curves are mutually consistent, this chain of (par) swaps cannot create or destroy PV. If they are inconsistent, the chain implies an arbitrageable discrepancy (or, in practice, a calibration failure that shows up as persistent “mis-marks” to the xccy/basis market).

### 21.2.3 Building an FX-Implied Foreign Curve

Given a strip of FX forwards and a domestic discount curve, we can construct the implied foreign discount curve point by point.

**Example Title**: FX-implied EUR discount curve from USD curve + EUR/USD forwards

**Context**
- You have a USD discount curve and a strip of EUR/USD forward quotes.
- You want an internally consistent EUR discount curve implied by the FX forwards.

**Timeline (Make Dates Concrete)**
- Valuation / trade date: 2026-02-03
- Spot-settlement date (assumption): 2026-02-05 (ignore holidays)
- Forward maturity dates: 2026-08-03 (6M), 2027-02-03 (1Y), 2028-02-03 (2Y)

**Inputs**
- Quote convention: $X$ and $F$ are USD per EUR.
- Spot: $X(0)=1.1000$ USD/EUR
- USD discount factors:
  - $P_d(0,0.5)=0.985112$, $P_d(0,1)=0.970446$, $P_d(0,2)=0.941765$
- Observed FX forwards:
  - $F(0,0.5)=1.111055$, $F(0,1)=1.122221$, $F(0,2)=1.144892$
- Day count / compounding: treat 6M/1Y/2Y as $T=0.5/1/2$; continuous compounding only when converting DFs to zero rates.

**Outputs (What You Produce)**
- FX-implied EUR discount factors $P_f(0,T)$ and EUR zero rates $r_f(T)$.
- FX delta of a par FX forward (per €1 notional): $\Delta_{FX}=P_f(0,T)$ USD per (USD/EUR).

**Step-by-step**
1. FX-implied foreign discount factors:
   $$P_f(0,T)=\frac{F(0,T)}{X(0)}P_d(0,T).$$
2. Convert implied discount factors into zero rates (optional):
   $$r_f(T)=-\frac{1}{T}\ln P_f(0,T).$$

**Implied EUR curve**
| $T$ | $F(0,T)/X(0)$ | $P_d(0,T)$ | $P_f(0,T)$ | $r_f(T)$ (cc) |
|---:|---:|---:|---:|---:|
| 0.5 | 1.010050 | 0.985112 | 0.995012 | 1.00% |
| 1.0 | 1.020201 | 0.970446 | 0.990050 | 1.00% |
| 2.0 | 1.040811 | 0.941765 | 0.980199 | 1.00% |

**Cashflows (per €1 notional, 1Y forward)**
| Date | Cashflow | Explanation |
|---|---:|---|
| 2027-02-03 | +€1 | receive foreign notional |
| 2027-02-03 | $-F(0,1)$ USD | pay domestic notional at the contracted forward |

**P&L / Risk Interpretation**
- A par FX forward has $PV=0$ at inception, but **FX delta is not zero**: $\Delta_{FX}=P_f(0,T)$ per €1.
- The “triangle check” is a quick build sanity test: domestic curve + FX forwards must imply a sensible foreign curve.

**Sanity Checks**
- Units: $F/X$ is dimensionless, so $P_f=(F/X)P_d$ is unitless. ✓
- Monotonicity: $P_f(0,0.5)>P_f(0,1)>P_f(0,2)$. ✓
- Reproduction: recompute $F = X P_f/P_d$ from the implied curve and recover the input forwards. ✓

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
>
> 2.  **Interest Rates**: Cost of Time (Domestic vs Foreign).
>
> 3.  **Forward Market**: Future Exchange.
>
> If you know any two, you *must* satisfy the third. If your curve construction uses Independent Domestic Swaps and Independent Foreign Swaps, it will almost certainly fail to match the Forward Market, breaking the triangle.

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

### 21.3.2 What the Market Quotes (Leg + Sign)

A common market convention is to quote a spread \(e\) on one leg of a cross-currency basis swap (often described in legacy IBOR terms such as “foreign floating + \(e\) vs USD floating flat”), but conventions vary. Always confirm the quote definition (which leg, pay/receive, sign, and index) before calibrating.

### 21.3.3 Why the Basis Can Be Non-Zero

If you build curves independently in each currency, the FX-forward-implied curve and the locally-bootstrapped curve generally do not coincide. The basis spread is the adjustment that reconciles them in traded instruments.

The reasons for the spread are twofold:
1. **Daily demand–supply imbalances**: there may be more demand for paying a LIBOR in a particular currency, which can lead to a positive basis spread.
2. **Corporate supply and hedging activity** (e.g., issuing in EUR and swapping cashflows into USD), which can create one-way basis demand.

Scale check: basis can be small in calm regimes and can move materially in stress. The practical implication is simple: treat the basis as a **market input** and a **risk factor**, not as a rounding error that “should” be zero.

### 21.3.4 Desk Reality Checklist

> **Desk Reality:** Basis is a traded risk factor and a curve input.
> **Common break:** Systems disagree because they assume different quote conventions (which leg has the spread) or different curve anchors (which curve is “discounting” vs “projection”).
> **What to check:** For any quoted basis point, write down (a) the two floating indices, (b) which leg carries the spread, (c) the notional-exchange convention, and (d) the collateral currency used for discounting.

---

## 21.4 Cross-Currency Basis Swaps: Constraints Beyond FX Forwards

In reality, the interbank FX forward market is rarely liquid beyond maturities of one year. Rather than relying on FX forwards, instead we can turn to the market for **floating-floating cross-currency (CRX) basis swaps**.

### 21.4.1 Structure of a Cross-Currency Basis Swap

Briefly speaking, CRX basis swaps exchange floating LIBOR payments in one currency for floating LIBOR payments in another currency, plus or minus a spread. The swaps involve an exchange of notionals at trade inception and at maturity; the ratio between the two notionals is normally set to equal the spot FX exchange rate prevailing at trade inception (exact conventions vary by pair and documentation).

**Key structural features:**

| Feature | Description |
|---------|-------------|
| **Floating legs** | Each currency pays its local LIBOR (or successor rate) |
| **Basis spread** | Added to one leg (convention varies) |
| **Notional exchange** | At inception and maturity, at spot FX ratio |
| **Maturities** | Quoted out to 30+ years |

**Critical insight:** A one-period CRX basis swap is identical to an FX forward contract. This establishes the continuity between short-dated FX forwards and long-dated basis swaps: the basis swap generalizes the FX forward to multiple periods with intermediate floating payments.

**Expand (one-period equivalence):** With only one accrual period, an xccy swap is essentially “exchange notionals + pay the funding on each notional.” If you PV each currency leg under its own curve and convert at spot, the par condition collapses to the FX-forward constraint. That is why short-dated FX forwards/FX swaps and one-period xccy structures all pin down the same triangle \(F = X\,P_f/P_d\); the differences are mostly conventions (spot lag, day count, which index accrues interest, and which leg carries any spread).

**Check (reduce to the FX-forward PV):** A forward that receives 1 foreign at \(T\) and pays \(K\) domestic has domestic PV \(V = X(0)P_f(0,T) - K P_d(0,T)\). Setting \(V=0\) gives \(K=F(0,T)=X(0)P_f(0,T)/P_d(0,T)\). One-period xccy par conditions are just this identity written with the instrument’s coupon conventions.

> **The Plumbing Diagram: Cashflows of a 5Y EUR/USD Xccy Basis Swap**
>
> Illustrative example: Party A (USD-based) receives EUR floating + basis, pays USD floating flat
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
> 3. **Spread-on-leg convention**: In this illustration, the basis is on the EUR leg; if $e=-30$bp, the EUR receiver gets (EUR floating) $-30$bp
> 4. **No FX risk on coupons during life**: Each coupon stays in its own currency

**Mark-to-Market (MTM) vs. Resettable Structures:**

> **Practitioner Note:** In modern markets, many long-dated xccy swaps use "mark-to-market" or "resettable" structures to reduce counterparty exposure.

In a standard xccy swap, the FX rate is locked at inception. If spot moves significantly, the PV mismatch creates large counterparty exposure.

**MTM xccy swap**: The notional exchange rate resets periodically (typically annually) to the prevailing spot rate. This reduces PFE (potential future exposure) and counterparty risk, but introduces additional cashflows to settle the notional revaluation.

Exact MTM/resettable mechanics vary by currency pair and documentation (reset frequency, whether notionals reset on one or both legs, and how the reset cashflow is settled). Treat the term sheet as the source of truth.

### 21.4.2 Valuation of a Cross-Currency Basis Swap

For a USD/JPY basis swap where the USD leg is floating flat in exchange for JPY floating plus spread $e_¥$, a common valuation form (in a simplified setting) is:

$$V_{\text{basisswap},\$}(0) = 1 - X(0) \times \left(\sum_{i=0}^{n-1}\left(\frac{P_¥^{(L)}(0,t_i)}{P_¥^{(L)}(0,t_{i+1})} - 1 + e_¥ \tau_i\right) P_¥(0,t_{i+1}) + P_¥(0,t_n)\right)$$

where:
- The USD floating leg plus notional repayment collapses to par (\$1) under the simplifying assumption that the USD index curve coincides with the USD discount curve
- The JPY leg is valued using forward JPY floating rates from $P_¥^{(L)}$ and discounted at $P_¥$

**Expand (what this expression is doing):** The bracketed term is the PV (in JPY, per unit JPY notional) of “foreign floating coupons + foreign principal,” where the one-period floating coupon is generated by the forward ratio \(P_¥^{(L)}(0,t_i)/P_¥^{(L)}(0,t_{i+1})-1\) and then discounted with \(P_¥(0,t_{i+1})\). The basis spread \(e_¥\) enters as an additive coupon shift \(e_¥\tau_i\) and therefore affects PV approximately linearly. Multiplying by spot \(X(0)\) converts the foreign-leg PV into USD; subtracting from 1 reflects “USD leg at par minus converted foreign-leg PV” under the stated simplifying assumptions.

**Checks (units + limits):** The forward ratio and \(e_¥\tau_i\) are dimensionless, \(P_¥\) is unitless, and the bracket is unitless; multiplying by \(X(0)\) (USD per JPY) produces USD per unit JPY notional. If \(e_¥=0\) and the curve set is internally consistent, the par swap should price near 0 (after consistent notional scaling and conventions). If you bump \(e_¥\) by +1 bp on the leg you receive, PV should move by roughly “foreign spread annuity × spot,” which is a quick sign/scale sanity check.

Market quotes a term structure of par basis spreads $e^{par}(T)$ across maturities (often out to 30Y+).

### 21.4.3 Why Basis Swaps Provide Curve Constraints

Failing to fit basis swaps creates the same kind of cross-market inconsistency as failing to fit FX forwards. A useful consistency check is the three-step conversion:

1. Swap fixed USD → USD floating (USD IRS)
2. Swap USD floating → JPY floating + $e$ (xccy basis swap)
3. Swap JPY floating + $e$ → fixed JPY (JPY IRS)

If your curve set is internally consistent, this chain of swaps cannot create or destroy PV. If it does, that shows up as persistent mispricing of xccy instruments (or “arbitrage” in a toy frictionless world).

> **Example 21.3: Solving for Par Basis Spread**
>
> **Setup:** 2-period toy basis swap (domestic USD, foreign JPY)
>
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

In fact, we have too many degrees of freedom: four curves, but only three separate markets to calibrate to. One way of handling this issue is to impose additional assumptions about the relationship between the discount curve $P$ and the pseudo-discount / projection curve $P^{(L)}$ in one chosen currency.

**Assumption 6.5.1 (historical):** In USD, the Libor pseudo-discount curve coincides with the real discount curve, i.e. $P_\$^{(L)}(t,T)=P_\$(t,T)$ for all $t$ and $T,\\,T \\ge t$.

Post-2007 collateralized discounting breaks this single-curve anchor; it is now generally accepted that the Libor rate is no longer a good proxy for a discounting rate on collateralized trades, so discounting and projection curves must be separated (see Chapter 20).

### 21.5.2 The Iterative Algorithm

One practical multi-currency curve construction procedure is:

**Step 1: Build USD curves**

Using standard bootstrapping from:
- Funding instruments (e.g., OIS/cash instruments for discounting)
- Index-referencing instruments (e.g., FRAs/swaps for the chosen projection index)

This gives the USD discount curve $P_\$(t)$ and the USD projection curve $P_\$^{(L)}(t)$.

**Step 2: Build foreign index curve $P_f^{(L)}(t)$**

From foreign-currency swaps, bootstrap $P_f^{(L)}(t)$ as if it were a discount curve.

**Step 3: Calibrate foreign discount curve $P_f(t)$ from basis swaps**

Represent $P_f$ as a multiplicative spread to $P_f^{(L)}$:

$$P_f(t) = P_f(T_i) \frac{P_f^{(L)}(t) e^{-\varepsilon_i(t - T_i)}}{P_f^{(L)}(T_i)}, \quad t \in [T_i, T_{i+1})$$

where $\varepsilon_i$ is a piecewise constant spread intensity.

Equivalently, in forward-rate form you can write the foreign discount forwards as the foreign projection forwards plus a spread function:

$$f_¥(t) = f_¥^{(L)}(t) + \varepsilon(t), \quad \varepsilon(t) = \sum_{i=0}^{N-1} \varepsilon_i \mathbf{1}_{\{t \in [T_i, T_{i+1})\}}$$

The constants $\varepsilon_0, \ldots, \varepsilon_{N-1}$ are calibrated by fitting to par-valued CRX basis swaps maturing at $T_1, \ldots, T_N$.

**Step 4: Iterate if necessary**

The initial bootstrap of $P_f^{(L)}$ treated it as a discount curve. With $P_f$ now calibrated, we can re-price the foreign benchmark instruments using the correct multi-curve valuation. Iterate steps 2-3 until convergence.

Iteration starts from the “single-curve” approximation ($P_f^{(L)}$ treated as discounting), then re-prices the foreign instruments under the calibrated discount/projection split until the pricing errors are below tolerance. In many cases this converges in only a few iterations because the initial approximation is close.

### 21.5.3 Numerical Walkthrough: The Iterative Algorithm in Action

> **Example 21.5: Complete Iterative Curve Construction**
>
> We build a simplified 2-point EUR discount curve using the Andersen-Piterbarg algorithm.
>
> **Given:**
>
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
> **Converged in 2-3 iterations** in this toy example.

> **Desk Reality: Why Iteration Converges Quickly**
>
> The initial guess (treating the projection curve as if it were a discount curve) can be close when the discount-vs-projection adjustment is modest, so the correction step is small.
>
> If the adjustment is large or the market inputs are noisy, you may need more iterations and tighter tolerances; always check fit and triangle consistency.

### 21.5.4 Avoiding Circularity

Forward FX and cross-currency basis swaps let you translate an index-discounting split chosen in one currency (say USD) into other currencies. However, to estimate this basis in USD (say), we need to rely on domestic markets only; doing otherwise will introduce a circularity into our arguments.

Practically: build USD discounting and USD projection curves from USD instruments (OIS plus the relevant index and basis markets; see Chapter 20), then use cross-currency instruments to infer foreign discounting adjustments.

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

The discussion so far has assumed standard collateral practices. In reality, the currency in which collateral is posted affects which discount curve applies. If a trade is *not* fully collateralized, then counterparty credit risk and funding effects can matter; handling those consistently is beyond pure curve construction.

### 21.6.1 The Collateral-Discount Curve Linkage

The fundamental principle: **discount at the collateral rate**.

In the case of perfect collateralization, valuation is obtained by discounting cash flows at the collateral rate $\tilde{c}_{t}$.

For a fully collateralized swap where collateral earns the overnight rate:
- **USD collateral** → discount at USD OIS (SOFR)
- **EUR collateral** → discount at EUR OIS (ESTR)
- **Multi-currency CSA** → more complex (optionality)

This linkage arises because the posted collateral earns the overnight rate. The counterparty receiving collateral funds the position at the same rate, making the overnight rate the appropriate discount rate.

### 21.6.2 What If Collateral Currency Differs from Trade Currency?

**Expand (two equivalent ways to PV):** If the trade pays USD cashflows but the CSA collateral currency is EUR, you can value a USD cashflow \(C_\$(T)\) either by:
1. Converting the future USD cashflow into EUR at the maturity-\(T\) FX forward \(F(0,T)\) (USD per EUR): \(C_\$(T)/F(0,T)\) EUR at \(T\); discount on the EUR curve; then convert the PV back to USD at spot \(X(0)\); or
2. Using an implied USD discount factor consistent with EUR collateral:
   $$P_{\$|\text{EUR coll}}(0,T)=\frac{X(0)}{F(0,T)}\,P_{€}(0,T).$$
Both views are the same identity rearranged; the important discipline is to use an FX forward curve that is consistent with the collateral regime (i.e., includes the relevant xccy basis inputs).

> **Example 21.6: Same Swap, Different Collateral, Different Value**
>
> Consider a 5Y USD interest rate swap (pay fixed, receive SOFR):
>
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

Pricing multi-currency CSA optionality requires modeling the **collateral currency option** (a cheapest-to-deliver feature) and therefore assumptions about basis dynamics/volatility. This is typically treated as part of XVA/CSA optionality rather than “just curve building.”

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

## 21.7 Risk Decomposition: FX Delta, Rates DV01, and Basis Exposure

Cross-currency instruments load on multiple risk factors that can move somewhat independently: spot FX, each currency’s curve(s), and the cross-currency basis. Risk systems typically report these as separate buckets so hedges and P&L explains don’t mix apples and oranges.

**Risk conventions used in this chapter**
- Basis point: $1\text{ bp}=10^{-4}$.
- FX quote: $X$ is domestic currency per 1 unit of foreign currency (same units for $F$).
- DV01 (per currency): **bump object** = the specified discounting zero-rate curve in that currency; **bump size** = rates **down** 1 bp; **DV01** := $PV(\text{rates down }1\text{bp})-PV(\text{base})$. Units: currency per 1bp. (So rates **up** 1bp gives $\Delta PV \approx -DV01$.)
- Basis01: **bump object** = the quoted par xccy basis spread $e$ on the stated leg; **bump size** = $e$ **up** 1 bp; **Basis01** := $PV(e+1\text{bp})-PV(e)$. Units: domestic currency per 1bp (after FX conversion where needed).
- “Hold fixed”: unless stated, when bumping one object (spot, a curve, or basis) hold the others fixed.

### 21.7.1 FX Delta

The sensitivity to spot FX changes. For an FX forward with PV = $X(0) P_f - K P_d$:

$$\frac{\partial V}{\partial X(0)} = P_f(0,T)$$

This measures how much PV changes (in domestic currency) for a unit change in the spot rate.

### 21.7.2 Interest Rate DV01 by Currency

For cross-currency portfolios, compute DV01 separately by currency (and be explicit about **which curve** you bump):
- **Domestic DV01 ($DV01_d$):** bump the domestic discounting curve (e.g., domestic OIS zero rates) **down** 1bp in parallel and reprice. Units: domestic currency per 1bp.
- **Foreign DV01 ($DV01_f$):** bump the foreign discounting curve (foreign OIS) **down** 1bp in parallel and reprice. Units: foreign currency per 1bp; if you report everything in domestic currency, convert by spot (or the desk’s reporting convention).

These are not fungible—you cannot hedge foreign rates DV01 with domestic rates instruments without also taking FX risk (unless you add an FX hedge).

### 21.7.3 Basis Exposure

Basis risk is the sensitivity to the quoted basis spread $e$ in xccy instruments. With the convention above:
- **Bump object:** the quoted par basis spread $e$ on the stated leg.
- **Bump size:** $+1$ bp.
- **Basis01:** $PV(e+1\text{bp})-PV(e)$ (domestic currency per 1bp, after FX conversion if needed).

For a position where $e$ enters valuation linearly to first order:

$$\frac{\partial V}{\partial e} = \text{Basis01}$$

This risk is distinct from both FX delta and rates DV01. A portfolio can be FX-hedged and DV01-neutral but still have significant basis exposure.

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

| Risk | Bump object | Bump size | Units | Sign convention (this chapter) |
|---|---|---:|---|---|
| FX delta $\Delta_{FX}$ | spot $X$ | derivative | domestic per (domestic/foreign) | $\Delta_{FX}=\partial PV/\partial X$ |
| Domestic $DV01_d$ | domestic discount curve zero rates | rates down 1bp | domestic currency per 1bp | $DV01_d := PV(\downarrow 1\text{bp})-PV$ |
| Foreign $DV01_f$ | foreign discount curve zero rates | rates down 1bp | foreign currency per 1bp | $DV01_f := PV(\downarrow 1\text{bp})-PV$ |
| Basis01 | quoted basis $e$ (stated leg) | $e$ up 1bp | domestic currency per 1bp | $Basis01 := PV(e+1\text{bp})-PV(e)$ |

### 21.7.5 P&L Attribution for Cross-Currency Portfolios

This book uses the DV01 convention: **DV01 is positive for positions that gain when rates fall**. Under that convention, the linear P&L approximation is:
$$\boxed{\Delta PV \approx \underbrace{\Delta_{FX} \cdot \delta S}_{\text{FX}} - \underbrace{DV01_d \cdot \delta r_d}_{\text{Dom Rates}} - \underbrace{DV01_f \cdot \delta r_f \cdot S}_{\text{For Rates}} + \underbrace{Basis01 \cdot \delta e}_{\text{Basis}} + \underbrace{\theta}_{\text{Carry}} + \underbrace{\varepsilon}_{\text{Unexplained}}}$$

where:
- $\Delta_{FX}$: FX delta
- $\delta S$: Change in spot FX rate
- $DV01_d$, $DV01_f$: Domestic and foreign DV01s (in $/bp and foreign-currency/bp)
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
> - USD DV01: $+\$45,000$ per bp (benefit from lower USD rates)
> - EUR DV01: $+€40,000$ per bp (benefit from lower EUR rates)
> - Basis01: $+\$4,500$ per bp (benefit from basis tightening)
>
> **Day's market moves:**
> - EUR/USD: 1.1000 → 1.1050 (ΔS = +0.0050, +0.45%)
> - USD 5Y: 4.00% → 4.05% (+5bp)
> - EUR 5Y: 3.00% → 2.95% (-5bp)
> - EUR/USD 5Y basis: -30bp → -25bp (+5bp tighter)
>
> **P&L attribution:**
> | Component | Calculation | P&L |
> |-----------|-------------|-----|
> | FX | $-€4.5mm \times 0.0050$ | $-\$22,500$ |
> | USD Rates | $-(+\$45,000) \times (+5)$ | $-\$225,000$ |
> | EUR Rates | $-(+€40,000) \times (-5) \times 1.1025$ | $+\$220,500$ |
> | Basis | $+\$4,500 \times 5$ | $+\$22,500$ |
> | **Total Explained** | | $-\$4,500$ |
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
> In crisis episodes, it is common to see a much larger “unexplained” residual than on normal days. This is a warning sign that the linear model is missing higher-order effects (basis convexity/cross-gamma) and that correlations/regimes have shifted.

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

FX quotes are directional. A quote “A/B = x” means **1 unit of A costs x units of B**.

Common examples:
- EUR/USD = 1.1000 means 1 EUR = 1.1000 USD.
- USD/JPY = 150.00 means 1 USD = 150.00 JPY.
- GBP/USD = 1.2500 means 1 GBP = 1.2500 USD.

**Impact on formulas:** If your market quote uses the inverse direction, you must invert spot and forward before applying CIP formulas. A common source of errors is misapplying the formula with an inverted quote.

### 21.9.2 Basis Swap Quoting Conventions

Quoting conventions differ by currency pair and venue. A common convention is to quote the spread on the non-USD leg, but you should **always confirm** the quote definition (which leg, pay/receive, sign) before trading or calibrating.

### 21.9.3 Settlement and Calendar Issues

Spot/settlement conventions are currency-pair specific (spot lag, holiday calendars, business day adjustments). For production systems, treat these as **contract inputs** and avoid hard-coding “typical” lags.

### 21.9.4 Common Implementation Errors

> **Pitfall — The Million-Dollar Sign Error:** Mixing up quote direction, basis sign, or notional exchange direction.
> **Why it matters:** A sign error on a large notional can create a multi-million PV error that looks like “basis” but is really just the wrong convention.
> **Quick check:** For any xccy instrument, write down (i) what is paid/received in each currency, (ii) the FX quote direction used, and (iii) which leg carries the basis spread.

**Common errors and how to avoid them:**

| Error | Symptom | Prevention |
|-------|---------|------------|
| Inverted FX quote | Forward points have wrong sign | Verify: higher rates → forward discount |
| Wrong basis leg | PV way off market | Check market convention for pair |
| Day count mismatch | Small but persistent errors | Verify both legs use correct convention |
| Settlement date misalignment | PV breaks on date transitions | Align to correct spot/fixing dates |
| Using wrong discount curve | PV differs from Street marks | Match collateral currency to CSA |

Avoid hard-coding “typical” basis levels or signs from memory. Use live market quotes, and always carry the quote definition (leg + sign) alongside the number.

---

## Summary

1. **CIP is the foundation**: $F(0,T)=X(0)\,P_f(0,T)/P_d(0,T)$ links spot FX, forward FX, and discount factors across currencies. Under consistent compounding assumptions, this is equivalent to the familiar rate form $F_0 = S_0 e^{(r_d - r_f)T}$.
2. **FX forwards constrain curves**: Given a domestic curve and FX forward quotes, the foreign curve is pinned down point-by-point via $P_f(0,T)=(F(0,T)/X(0))P_d(0,T)$. (Triangle check: domestic curve + forwards $\Rightarrow$ foreign curve.)
3. **Cross-currency basis is a reconciliation spread**: Basis swap quotes adjust one leg so traded xccy instruments clear given the curve set. Quote conventions (which leg carries the spread; sign) matter as much as the number.
4. **Basis swaps extend the constraint horizon**: FX forwards tend to be liquid at short maturities; longer-dated constraints come from xccy basis swaps. A one-period xccy basis swap is the same economic object as an FX forward.
5. **Multi-currency curve construction is anchored**: Build an anchor currency’s discounting/projection curves from domestic instruments, then use FX forwards + basis swaps to infer the linked foreign curve(s), iterating only as needed for consistency.
6. **Collateral currency matters**: With collateralized pricing, discounting is tied to the collateral remuneration rate; if collateral currency differs from payout currency, cross-currency curves/basis are part of the translation.
7. **Risk decomposes cleanly**: FX delta ($\partial PV/\partial X$), DV01 by currency (explicit bump object + sign), and Basis01 are distinct risk dimensions and appear separately in P&L explain.
8. **Most production breaks are conventions**: quote direction, basis-leg/sign, day-count/schedule generation, and mixing projection vs discount curves are the common sources of “mystery basis” and mis-marks.

---

## Key Concepts

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **Covered interest parity (CIP)** | $F(0,T)=X(0)\,P_f(0,T)/P_d(0,T)$ (with consistent quote direction) | Links FX quotes to curve constraints |
| **FX-implied curve** | Foreign discount factors derived from a domestic curve + FX forwards | Enforces cross-currency consistency (triangle check) |
| **Cross-currency basis spread ($e$)** | Quoted spread on one leg of a floating–floating xccy swap that makes PV = 0 given the curve set | Market input that reconciles curves; separate risk factor |
| **Basis01** | $PV(e+1\text{bp})-PV(e)$ with the stated quote convention | Hedge sizing and P&L explain for basis moves |
| **Forward FX rate** | $X_T(t)=X(t)\,P_f(t,T)/P_d(t,T)$ | Replication and valuation using ZCBs |
| **Discount vs projection curves** | Discount at the collateral/OIS curve; project via index curves | Avoids silent “single-curve” assumptions in multi-curve pricing |
| **Notional exchange** | Exchange of principals at inception/maturity (or periodically in resettable structures) | Drives FX delta and exposure profile |
| **Anchor currency / circularity** | Choose an anchor curve set from domestic instruments | Avoids circular calibration across currencies |
| **DV01 by currency** | DV01 computed per curve (per currency), then converted to reporting currency if needed | Risk decomposition across FX + rates + basis |

---

## Notation

| Symbol | Meaning | Units / Convention |
|---|---|---|
| $d, f$ | domestic and foreign currencies | n/a |
| $X(0)$ | spot FX | domestic currency per 1 foreign currency |
| $F(0,T)$ | FX forward (maturity $T$) | domestic currency per 1 foreign currency |
| $P_d(0,T), P_f(0,T)$ | discount factors | unitless; $P(T,T)=1$ |
| $P^{(L)}(0,T)$ | pseudo-discount / projection curve for an index | unitless; used to generate forwards |
| $\tau(t_i,t_{i+1})$ | accrual year fraction | years; depends on day-count convention |
| $r_d(T), r_f(T)$ | (zero) rates corresponding to $P_d, P_f$ | per year; compounding must be stated if used |
| $e$ | quoted xccy basis spread | per year (often quoted in bp) on the stated leg |
| $s(t)$ | (yield) spread between pseudo and true discounting | per year; $P^{(L)}(t)=P(t)e^{-s(t)t}$ in a simple parameterization |
| $\Delta_{FX}$ | FX delta | domestic currency per (domestic/foreign); $\Delta_{FX}=\partial PV/\partial X$ |
| $DV01_d, DV01_f$ | DV01 by currency | currency per 1bp; $DV01 := PV(\downarrow 1\text{bp})-PV$ for stated bump object |
| $Basis01$ | basis sensitivity | domestic currency per 1bp; $Basis01 := PV(e+1\text{bp})-PV(e)$ |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What does CIP relate? | Spot FX, forward FX, and discount factors across two currencies |
| 2 | CIP in discount-factor form? | $F(0,T)=X(0)\,P_f(0,T)/P_d(0,T)$ |
| 3 | Solve for the FX-implied foreign discount factor? | $P_f(0,T)=(F(0,T)/X(0))\,P_d(0,T)$ |
| 4 | What is an “FX-implied curve”? | A foreign discount curve implied by a domestic curve plus a strip of FX forwards |
| 5 | What is a (floating–floating) xccy basis swap? | Exchange floating payments in two currencies plus a spread; typically includes notional exchanges |
| 6 | Why are basis swaps used for long maturities? | FX forwards tend to be liquid at the short end; basis swaps provide longer-dated constraints |
| 7 | Common quoting pitfall for basis? | Which leg carries the spread (and its sign) can vary by market—always confirm |
| 8 | What is Basis01? | $Basis01 := PV(e+1\text{bp})-PV(e)$ for the quoted basis $e$ (units: domestic currency per 1bp) |
| 9 | DV01 convention used in this book? | $DV01 := PV(\text{rates down }1\text{bp})-PV(\text{base})$ for the stated bump object |
| 10 | DV01 bump object in this chapter (by default)? | The discounting zero-rate curve in that currency (often OIS), bumped down 1bp in parallel |
| 11 | Basis01 bump object? | The quoted par basis spread $e$ on the stated leg, bumped up 1bp |
| 12 | What is the “triangle check”? | Any two of {domestic curve, foreign curve, FX forwards} determine the third; mismatch signals inconsistency |
| 13 | Why can two systems disagree on “basis”? | Different quote conventions, different curve anchors, or different collateral/discounting assumptions |
| 14 | How does collateral currency affect discounting? | Discounting is tied to collateral remuneration; multi-currency CSA adds optionality |
| 15 | What are the main xccy risk buckets? | FX delta, DV01 by currency, and Basis01 |
| 16 | FX delta of a par FX forward (receive 1 foreign at $T$)? | $\Delta_{FX}=P_f(0,T)$ (per 1 unit of foreign notional) |
| 17 | One-period xccy basis swap equals what? | An FX forward (same economic constraint) |
| 18 | Most common implementation failure mode? | Quote direction / sign / leg mismatch (“basis” that is really a convention error) |
| 19 | What does “hold fixed” mean in bump risk? | When bumping one factor (spot/curve/basis), keep the others unchanged |
| 20 | Quick CIP sanity check (domestic-per-foreign quote)? | If domestic rates exceed foreign rates, you should typically see $F(0,T) > X(0)$ |

---

## Mini Problem Set

1. **(Compute)** Given $S(0) = 1.25$ USD/EUR, $r_d = 4\%$, $r_f = 1.5\%$, $T = 1$ (continuous), compute $F(0,1)$.
2. **(Compute)** Given $X(0) = 1.10$, $P_d(0,2) = 0.94$, $P_f(0,2) = 0.98$, compute $F(0,2)$.
3. **(Compute)** Derive $P_f(0,T)$ from $X(0)$, $F(0,T)$, and $P_d(0,T)$.
4. **(Compute)** An FX forward receives 1 EUR at $T$ and pays $K$ USD. Write the PV at time 0 and solve for $K$ that makes PV = 0.
5. **(Compute)** Using $P^{(L)}(t) = P(t) e^{-s(t)t}$, solve for $s(t)$ given $P^{(L)}(1) = 0.9885$ and $P(1) = 0.9900$.
6. **(Concept)** Explain why building USD and EUR curves independently can create inconsistencies with FX forwards and basis swaps.
7. **(Compute)** Build a 3-point FX-implied EUR curve from: $P_d(0.5)=0.99$, $P_d(1)=0.97$, $P_d(2)=0.93$, $X=1.15$, $F(0.5)=1.16$, $F(1)=1.18$, $F(2)=1.22$. Verify monotonicity.
8. **(Proof/Compute)** In a two-curve setup, show that if $P^{(L)}=P$, the PV of a floating leg with principal repayment telescopes to par.
9. **(Desk)** If a market forward is far from the CIP-implied forward, outline an arbitrage (or, in practice, which constraint/assumption must be wrong).
10. **(Compute — P&L Attribution)** A 5Y EUR/USD xccy swap has: FX delta = $-€5\text{mm}$, USD $DV01_d=+\$50\text{k/bp}$, EUR $DV01_f=+€45\text{k/bp}$, $Basis01=+\$5\text{k/bp}$. Over one day: EUR/USD moves 1.10 → 1.12, USD rates +3bp, EUR rates −2bp, quoted basis $e$ moves −30bp → −28bp. Attribute the P&L.
11. **(Desk/Compute)** A corporate can issue USD at 4.90% or EUR at 3.70%. EUR 5Y swap rate is 3.00%, USD 5Y swap rate is 4.00%, and basis is −25bp on the EUR leg. Compute a toy “all-in” USD funding cost via EUR issuance + swap (state what you are assuming).
12. **(Concept/Compute)** A 5Y USD IRS has $DV01 \approx 48{,}000\,\$/\text{bp}$. If collateral currency changes from USD to EUR and the relevant cross-currency adjustment is about 30bp, estimate the PV impact and list what you would need for a full valuation.

### Solution Sketches (Selected)
1. $F = 1.25\,e^{(0.04-0.015)\cdot 1} \approx 1.2817$ USD/EUR.
2. $F = 1.10\times (0.98/0.94) \approx 1.1468$.
5. $s(1) = -\ln(0.9885/0.9900) \approx 0.00152 \approx 15.2\text{ bp}$.
6. If you ignore FX forwards/basis swaps, the domestic- and foreign-bootstrapped curves generally won’t satisfy $F=X\,P_f/P_d$. The inconsistency shows up as persistent mis-pricing of xccy instruments (or “arbitrage” in a toy world with no frictions).
10. Use
   $$\Delta PV \approx \Delta_{FX}\,\Delta S - DV01_d\,\Delta r_d - DV01_f\,\Delta r_f\,S + Basis01\,\Delta e,$$
   with $S\approx 1.11$:
   - FX: $-€5\text{mm}\times 0.02 = -\$100{,}000$
   - USD rates: $-(+\$50\text{k})\times (+3) = -\$150{,}000$
   - EUR rates: $-(+€45\text{k})\times (-2)\times 1.11 = +\$99{,}900$
   - Basis: $(+\$5\text{k})\times (+2)=+\$10{,}000$
   - Total: $\approx -\$140{,}100$
11. Toy assumption: treat “swap to USD” as adding USD swap rate, plus EUR credit spread, plus basis benefit on the EUR leg. All-in $\approx 4.00\% + 0.70\% - 0.25\% = 4.45\%$ vs 4.90%.
12. Back-of-envelope: $30\text{bp}\times (48{,}000\,\$/\text{bp})\approx \$1.44\text{mm}$. Full valuation needs the CSA terms (eligible collateral + remuneration), the cross-currency curve/basis model used for discounting translation, and the trade’s exact cashflow schedule.

---

## References

- Andersen & Piterbarg, *Interest Rate Modeling*, “Notations and FX Forwards”; “Cross-Currency Curve Construction”; “Cross-Currency Basis Swaps”; collateral/OIS discounting discussion.
- Neftci, *Principles of Financial Engineering*, “Covered interest rate parity / FX swap engineering”; “Cross-currency swap spreads and valuation”.
- Hull, *Options, Futures, and Other Derivatives*, “Interest Rate Parity” and FX forwards; “Currency Swaps” (and OIS discounting discussion where applicable).
- Tuckman & Serrat, *Fixed Income Securities: Tools for Today’s Markets*, currency swaps and fixed-income risk conventions (DV01 sign/units).
- Crépey, *Counterparty Risk and Funding*, “Collateral” (CSA mechanics; cash collateral remunerated at an OIS rate in simplified frameworks).
- Brigo et al., *Counterparty Credit Risk, Collateral and Funding*, “Pricing with two currencies: foundations”; cross-currency swaps and basis in a collateral/multi-curve setting.
