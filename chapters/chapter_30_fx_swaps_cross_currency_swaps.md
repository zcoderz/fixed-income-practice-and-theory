# Chapter 30: FX Swaps and Cross-Currency Swaps — Structure and Valuation Dependencies

---

## Learning Objectives
- Translate FX swap and cross-currency swap quotes into explicit cashflows (with correct units and timelines).
- Price FX forwards/swaps using covered interest parity (CIP) in discount-factor form.
- Price a cross-currency basis swap as “domestic leg PV − spot × foreign leg PV” with explicit discounting vs projection curves.
- Compute and interpret FX delta, curve DV01s, and basis DV01 with explicit bump objects, units, and sign conventions.
- Recognize common pitfalls: quote direction, forward points sign, value dates/calendars, and inconsistent curve/FX inputs.

## Introduction

Prerequisites: [Chapter 29](chapters/chapter_29_fx_spot_forwards.md), [Chapter 25](chapters/chapter_25_interest_rate_swaps_mechanics_valuation.md), [Chapter 26](chapters/chapter_26_swap_pv01_dv01_hedging.md), [Chapter 28](chapters/chapter_28_basis_trades.md)

Follow-on: [Chapter 31](chapters/chapter_31_multi_currency_risk.md), [Chapter 33](chapters/chapter_33_collateral_discounting_ois.md), [Chapter 32](chapters/chapter_32_counterparty_exposure_basics.md), [Chapter 34](chapters/chapter_34_xva_overview.md)

An **FX swap** is a short-dated funding trade: you exchange notionals on the spot date and reverse the exchange at a pre-agreed forward rate. A **cross-currency swap** (XCCY) extends the idea across multiple payment dates by exchanging interest payments in two currencies (and typically exchanging notionals at inception and maturity). In a **cross-currency basis swap**, one floating leg includes an extra spread $b$ so the swap is par at inception (or, equivalently, the spread is accompanied by an upfront payment).

This chapter uses a desk-friendly pipeline:

1. **Quote → contract terms:** spot $X(0)$, forward $F(0,T)$ or forward points, basis spread $b$, calendars/value dates.
2. **Contract terms → cashflows:** notional exchanges and floating coupons $N \cdot (L+b)\cdot \tau$.
3. **Cashflows → PV:** discount each currency’s cashflows on the appropriate discount curve and convert using spot/forwards consistently.
4. **PV → risk:** FX delta, curve DV01s, and basis DV01 (with explicit bump objects and units).

Out of scope (by design): building full cross-currency curves (see [Chapter 21](chapters/chapter_21_cross_currency_curves.md)); and full CSA/XVA treatment (see [Chapter 33](chapters/chapter_33_collateral_discounting_ois.md) and [Chapter 34](chapters/chapter_34_xva_overview.md)).

---

## 30.1 FX Forwards and Covered Interest Parity

### 30.1.1 The Forward FX Rate

An FX forward contract sets a future exchange of currencies at a pre-agreed rate $F(0,T)$. Under no-arbitrage, the forward rate is pinned down by interest rate differentials (not by “where spot will be”).

In continuously-compounded form (domestic currency $d$, foreign currency $f$):

$$\boxed{F_0 = S_0 e^{(r - r_f)T}}$$

where $S_0$ is the spot rate (domestic per 1 foreign), $r$ is the domestic continuously-compounded rate to $T$, and $r_f$ is the foreign rate to $T$.

In discount-factor form (recommended for curve-based work):

$$\boxed{F(0,T) = X(0) \frac{P_f(0,T)}{P_d(0,T)}} \tag{CIP-DF}$$

This is covered interest parity (CIP) in discount-factor form. The relationship arises because converting currencies via the forward market must produce the same value as converting via the spot market and investing the proceeds.

**Intuition:** If domestic interest rates exceed foreign rates ($P_d \lt P_f$ for the same maturity), then $F \gt X$—the foreign currency trades at a forward premium. This compensates a domestic investor for holding the lower-yielding foreign currency: the forward rate locks in an appreciation that offsets the interest differential.

**Unit check:** $P_f/P_d$ is dimensionless; $X(0)$ has units domestic/foreign; hence $F(0,T)$ has units domestic/foreign. ✓

> **Desk Reality:** FX forwards are often quoted as **forward points** (swap points) rather than as an outright forward. Quoting points helps separate the risks associated with interest-rate differentials and spot exchange-rate movements.
> **Common break:** Points may be displayed with an implicit sign or with pair-specific quoting conventions. Always convert points → outright $F$ before computing PV/risk.
> **What to check:** Verify $F(0,T)/X(0) \approx P_f(0,T)/P_d(0,T)$ for the same maturity and quote direction.

> **Pitfall — Quote direction and points sign:** Some pairs are quoted as “USD per EUR” while others are “JPY per USD”, and forward points may be applied with a market-specific sign convention.
> **Why it matters:** Flipping the quote or mis-applying points can change PV and hedge ratios by large factors.
> **Quick check:** Express everything as “domestic per 1 foreign” and confirm $F/X$ lines up with $P_f/P_d$.

### 30.1.2 Present Value of an FX Forward

Consider a forward where at time $T$ you receive 1 unit of foreign currency and pay $K$ units of domestic currency. The time-0 present value in domestic currency is:

$$\boxed{PV_0^{(d)} = X(0) P_f(0,T) - K P_d(0,T)} \tag{FXF-PV}$$

**Derivation:** "Receive 1 foreign at $T$" is economically equivalent to holding a foreign zero-coupon bond paying 1 at $T$, worth $P_f(0,T)$ in foreign terms; converting at spot gives domestic value $X(0) P_f(0,T)$. "Pay $K$ domestic at $T$" is a domestic ZCB liability of PV $K P_d(0,T)$. The net is their difference.

**At-market forward:** Setting $PV_0 = 0$ and solving for $K$:

$$K^* = X(0) \frac{P_f(0,T)}{P_d(0,T)} = F(0,T)$$

which confirms the CIP formula. (A fixed-for-fixed currency swap can be viewed as a portfolio of FX forwards—one for each exchange date.)

---

## 30.2 FX Swaps: Short-Dated Funding Trades

### 30.2.1 Structure: Spot Plus Forward

An FX swap combines two legs:
- **Near leg:** Exchange notionals at spot rate $X(0)$
- **Far leg:** Reverse the exchange at a pre-agreed forward rate $F(0,T)$

For example, a bank with euros that needs dollars might enter the following transaction:

*Near leg (time 0):*
- Pay €100 million, receive USD 110 million at spot $X(0) = 1.10$ USD/EUR

***Far leg (time $T=1$):***
- Pay USD 111.5 million, receive \mathrm{EUR}\,100 million at forward $F(0,1) = 1.115$

From the perspective of the USD receiver, the trade is “borrow USD / lend EUR”: you receive USD now and repay USD at $T$, while paying EUR now and receiving EUR at $T$.

### 30.2.2 From Spot/Forward to an Implied Funding Rate

For short maturities, it is often useful to translate the FX swap quote into an **implied money-market rate**.

Let $\tau$ be the year fraction from the near-leg settlement date to maturity under the relevant money-market day count. Suppose the foreign simple rate over $[0,T]$ is $r_f$. Then the domestic simple rate $r_d$ implied by the FX swap satisfies:

$$\boxed{1+r_d\,\tau=\frac{F(0,T)}{X(0)}\left(1+r_f\,\tau\right)} \tag{FXSWAP-IMPL}$$

**Check (units and toy numbers):** $F/X$ is dimensionless, and $(1+r\,\tau)$ is a dimensionless “growth factor,” so the identity is unit-consistent. Solving explicitly,
$$r_d=\frac{1}{\tau}\left(\frac{F}{X}(1+r_f\,\tau)-1\right).$$
For example, if $\tau=1$, $r_f=2\%$, and $F/X=1.1150/1.10$, then $r_d\approx 3.38\%$ (as in Example B). The key idea is translation: forward points are just an interest differential expressed in FX units once day counts/compounding are made consistent.

When CIP holds and the discount factors are consistent, this relationship lines up with the interest differential embedded in $P_d$ and $P_f$. When it does not (after making conventions consistent), the difference is often described as a **cross-currency basis**.

### 30.2.3 FX Swap vs. Cross-Currency Swap: A Detailed Comparison

A single-period FX swap is economically very close to an FX forward (and a one-period floating–floating cross-currency basis swap is equivalent to an FX forward). The differences matter once you move beyond one period.

| Feature | FX Swap | Cross-Currency (Basis) Swap |
|---------|---------|---------------------------|
| **Horizon (typical)** | Liquid out to ~1y in interbank markets | Used for multi-year horizons when FX forwards are illiquid |
| **Structure** | One near notional exchange + one far notional exchange | Periodic floating coupons in two currencies; basis $b$ on one leg; notionals often exchanged at inception and maturity |
| **Main quote** | Outright forward or forward points | Basis spread $b$ (and which leg it applies to) |
| **Primary risks** | FX spot + forward points/basis | FX spot + domestic and foreign curves + basis |

### 30.2.4 Single-Period vs Multi-Period Economics

For a single period, FX swap and XCCY swap economics are identical. Consider:

- **FX Swap:** Pay USD 100m at T=0, receive $100 \times (1 + r_{\mathrm{USD}} \times T)$ at T=1 (implicitly through forward rate)
- **XCCY Swap (1 period):** Pay USD 100m at T=0, receive USD floating at T=1, exchange principals at maturity

Both achieve the same economic transfer. However, for multi-period swaps:

- **Rolled FX swaps:** Each rollover re-fixes forward points and funding conditions.
- **Multi-period XCCY:** The schedule and basis spread are locked at inception (given curve/CSA assumptions).

This “roll” risk is one reason long-dated hedgers often prefer multi-period cross-currency swaps to repeated short-dated rolls.

**Check (roll-cost scaling):** Rolling is economically “re-fixing” your funding spread. If the implied funding differential moves by 10 bp between rolls, then on a USD 100m notional the cash impact over a quarter is roughly $100 \times 10\text{ bp} \times 0.25 = 25{,}000$ USD (ignoring compounding). That is small per roll but meaningful when leveraged or when basis moves are large in stress.

---

## 30.3 Cross-Currency Swaps and the Basis

### 30.3.1 The Basic Structure

A cross-currency swap exchanges cashflows in two currencies. In the common floating–floating **basis** form, the parties exchange floating coupons in two currencies and (typically) exchange notionals at inception and maturity. One leg may include a spread $b$ (the basis).

The canonical structure for a cross-currency basis swap:
- **At inception:** Exchange notionals at spot (e.g., pay USD 100m USD, receive \mathrm{EUR}\,90.9m at $X(0)=1.10$)
- **During the life:** Party A pays USD floating (e.g., SOFR); Party B pays EUR floating (e.g., EURIBOR) **plus or minus a spread** $b$
- **At maturity:** Re-exchange the same notionals

**Cashflow sign convention (one side of the trade):**

To make signs concrete, take domestic = USD, foreign = EUR, and a position that **receives USD floating** and **pays EUR floating plus basis** (with notionals exchanged at inception and maturity). Cashflows are:

| Date | USD leg (receive) | EUR leg (pay) |
|---|---:|---:|
| Inception | $+N_{\mathrm{\mathrm{USD}}}$ | $-N_{\mathrm{EUR}}$ |
| Each payment date $t_{i+1}$ | $+N_{\mathrm{\mathrm{USD}}}\,L_{\mathrm{\mathrm{USD}}}(t_i,t_{i+1})\,\tau_i$ | $-N_{\mathrm{EUR}}\,(L_{\mathrm{EUR}}(t_i,t_{i+1})+b)\,\tau_i$ |
| Maturity | $+N_{\mathrm{\mathrm{USD}}}$ | $-N_{\mathrm{EUR}}$ |

The other side has the opposite signs. Some contracts omit notional exchanges, include stubs, or place the basis on the other leg—always confirm the exact payoff definition.

### 30.3.2 The Cross-Currency Basis Spread

The **basis** $b$ is the spread added to one floating leg to make the trade par at inception (or to match an agreed upfront payment). Conceptually, it is the price of swapping funding between currencies once you account for curve and collateral assumptions.

In this chapter’s notation, we write the foreign floating coupons as $N_f (L_f + b)\tau$. From the perspective of a payer of the foreign leg, increasing $b$ increases outflows and reduces PV.

Sometimes the basis is also described via a “CRX yield spread” $s(t)$ linking a projection (index) curve to a discount curve through a relation of the form $P^{(L)}(t)=P(t)e^{-s(t)t}$. (This is a modeling convenience; the key tradable object is still the quoted spread $b$.)

> **Pitfall — Which leg gets the basis:** Market quotes may state “USD flat vs foreign $+$ basis” or vice versa.
> **Why it matters:** A sign/leg mix-up flips the economics and the basis DV01 sign.
> **Quick check:** Write the cashflow formula for each leg and confirm: “if $b$ goes up by 1 bp, do I pay more or receive more?”

### 30.3.3 Why the Basis Exists: Funding, Collateral, and Multi-Curve Effects

In a frictionless single-curve world, covered interest parity would tightly link FX forwards to the two money-market curves and “basis” would be near zero. In practice, two effects matter:

1. **Multi-curve effects:** once projection and discounting curves differ (e.g., discounting collateralized payoffs on OIS while projecting IBOR-style cashflows), a floating leg need not price exactly at par, and the mismatch differs by currency. This shows up as a basis spread in cross-currency swaps.
2. **Funding/credit and balance-sheet frictions:** even if an apparent arbitrage exists, constraints and funding spreads can prevent the basis from being fully traded away, allowing persistent deviations.

---

## 30.4 Valuation Framework: Project and Discount in Each Currency

### 30.4.1 The Fundamental Principle

Value each leg in its own currency, discount on the appropriate curve, and convert using the spot/forward FX rate consistently.

For a swap where you receive domestic floating and pay foreign floating plus basis:

$$\boxed{PV^{(d)} = PV_d^{(d)} - X(0) \cdot PV_f^{(f)}(b)}$$

where $PV_d^{(d)}$ is the domestic-currency PV of the domestic leg (discounted on the domestic curve), and $PV_f^{(f)}(b)$ is the foreign-currency PV of the foreign leg (discounted on the foreign curve, including the basis spread).

### 30.4.2 Separating Projection and Discounting

Modern multi-curve frameworks distinguish between:
- **Discount curves** $P_d$, $P_f$: Used to present-value known cashflows
- **Projection curves** $P_c^{(L)}$: Used to forecast floating rates via:

$$L_c(0, t_i, t_{i+1}) = \frac{1}{\tau_i}\left(\frac{P_c^{(L)}(0, t_i)}{P_c^{(L)}(0, t_{i+1})} - 1\right)$$

This separation matters whenever the floating index embeds credit/liquidity premia (so it should not be used as a “risk-free” discount rate) and whenever collateralized discounting is tied to an overnight collateral rate.

### 30.4.3 The Full XCCY PV Formula

One concrete example (USD/JPY basis swap, PV reported in USD): receive USD floating flat and pay JPY floating plus a spread $b$, with notionals exchanged at inception (at spot) and re-exchanged at maturity.

To keep notation compact, the expression below suppresses explicit notionals. In practice you compute each leg PV in its own currency using its contractual notional (e.g., $N_{\mathrm{\mathrm{USD}}}$ and $N_{\mathrm{JPY}}$), then convert the JPY PV into USD using the spot $X(0)$ under the chapter quote direction (domestic per 1 foreign).

$$\boxed{
\begin{aligned}
V_{\text{basisswap},\mathrm{\mathrm{USD}}}(0) &= \sum_{i=0}^{n-1} L_{\mathrm{\mathrm{USD}}}(0; t_i, t_{i+1}) \,\tau_i\, P_{\mathrm{\mathrm{USD}}}(0, t_{i+1}) + P_{\mathrm{\mathrm{USD}}}(0, t_n) \\
&\quad - X(0) \left( \sum_{i=0}^{n-1} \big(L_{\mathrm{JPY}}(0; t_i, t_{i+1}) + b\big) \,\tau_i\, P_{\mathrm{JPY}}(0, t_{i+1}) + P_{\mathrm{JPY}}(0, t_n) \right)
\end{aligned}
} \tag{XCCY-PV}$$

If the USD projection curve equals the USD discount curve, the PV of the USD floating leg (including the final notional payment) is approximately 1 *per unit USD notional* at inception; the non-trivial part of the valuation is then the FX-converted foreign leg (plus basis).

**Check (single-curve limiting case):** In an idealized single-curve world where each floating index is projected and discounted on the same curve and $b=0$, each “float + final notional” leg prices to par (≈ 1 per unit notional). If notionals are spot-matched at inception (e.g., $N_{\mathrm{\mathrm{USD}}} = X(0)\,N_{\mathrm{JPY}}$ under USD-per-JPY quoting), the two par legs cancel at inception and the swap is near zero PV. A non-zero par basis $b$ in practice comes from curve/collateral choices and frictions, not from the mechanical notional exchanges.

**What each curve does:**

| Curve | Role |
|-------|------|
| $P_{\mathrm{\mathrm{USD}}}(0,T)$ | Discounts USD cashflows |
| $P_{\mathrm{JPY}}(0,T)$ | Discounts JPY cashflows |
| $P_{\mathrm{JPY}}^{(L)}(0,T)$ | Generates forward JPY rates via ratio formula |
| $X(0)$ | Converts foreign-currency PV into domestic PV |
| $b^{\text{par}}$ | Quoted spread that sets $V = 0$ (given curves) |

### 30.4.4 FX Conversion Consistency

Under CIP, there is an equivalence between two valuation approaches for a foreign cashflow $C_f(T)$:

**Approach A:** Foreign discount + spot conversion
$$PV^{(d)} = X(0) \cdot P_f(0,T) \cdot C_f(T)$$

**Approach B:** Forward FX + domestic discount
$$PV^{(d)} = P_d(0,T) \cdot F(0,T) \cdot C_f(T)$$

These are identical when $F(0,T) = X(0) \frac{P_f(0,T)}{P_d(0,T)}$:

$$P_d(0,T) \cdot F(0,T) = P_d(0,T) \cdot X(0) \frac{P_f(0,T)}{P_d(0,T)} = X(0) \cdot P_f(0,T)$$

**Why this matters:** If your inputs are internally consistent (discount factors, FX forwards, and any cross-currency basis quotes), both approaches should agree. If they do not, you either have (i) inconsistent curve/FX inputs, or (ii) a deliberate “risk decomposition” choice where you hold some objects fixed while bumping others. In production systems, large mismatches usually show up as valuation breaks between desks/systems.

### 30.4.5 Collateral Currency and Discount Curve Selection

Collateralization affects discounting. In the case of *perfect* collateralization (idealized continuous margining with cash collateral), valuation can be obtained by discounting cashflows at the collateral remuneration rate. The precise multi-currency implementation depends on CSA terms and modeling choices; see [Chapter 33](chapters/chapter_33_collateral_discounting_ois.md) for a full treatment.

| CSA Collateral | Discount Curve | Example Application |
|---------------|----------------|---------------------|
| USD cash | USD OIS curve (e.g., SOFR) | EUR/USD swap, USD-collateralized |
| EUR cash | EUR OIS curve (e.g., €STR) | EUR/USD swap, EUR-collateralized |
| No collateral | Funding/credit adjustments needed | Uncollateralized pricing is not “single-curve” |

> **Desk Reality: CSA Terms as a Data Dependency**
>
> Two systems can produce different PVs for the same XCCY trade if they disagree on CSA inputs (collateral currency, remuneration rate, margining frequency, thresholds). Before debugging curves, confirm the CSA specification each system is using.

---

## 30.5 Worked Examples

### Example Conventions

Throughout these examples:
- Quote direction: $X =$ USD per EUR (domestic = USD, foreign = EUR)
- Rate conversion: Toy simple annual compounding $P(0,T) = 1/(1 + rT)$ (labeled as toy, not a market convention claim)
- Notional exchange: Included at maturity; initial exchange at spot has zero PV at inception

### 30.5.1 Worked Example: 1Y EUR/USD FX Swap → Cashflows → PV → Risk

**Context**
- You are EUR-funded but need USD for one year. You use an FX swap to exchange EUR for USD on the spot date and reverse the exchange at maturity.

**Timeline (illustrative; holiday adjustments ignored)**
- Trade date: 2026-02-17
- Spot/value date (near leg settlement): 2026-02-19 (T+2)
- Maturity/value date (far leg settlement): 2027-02-19

**Inputs**
- Quote direction: $X$ is USD per EUR (domestic = USD, foreign = EUR).
- Spot: $X(0)=1.10$ USD/EUR.
- 1Y forward quote: $F_{\mathrm{mkt}}(0,1)=1.1150$ USD/EUR (equivalently, forward points of $+0.0150$).
- Notional: $N_f=\mathrm{EUR}\,10{,}000{,}000$; implied USD notional $N_d=X(0)N_f=\mathrm{USD} 11{,}000{,}000$.
- Discount factors (toy inputs for illustration): $P_{\mathrm{\mathrm{USD}}}(0,1)=0.98$, $P_{\mathrm{EUR}}(0,1)=0.99$.

**Outputs**
- CIP-fair forward $F^{\star}(0,1)$ (given $P_{\mathrm{\mathrm{USD}}},P_{\mathrm{EUR}}$).
- PV in USD of the far-leg exchange at the quoted forward.
- FX delta (units: EUR; PV sensitivity to spot $X$).

**Step-by-step**
1. **Translate quote → cashflows (from the USD receiver’s perspective):**
   - Near leg (2026-02-19): receive USD 11.0m, pay EUR 10.0m.
   - Far leg (2027-02-19): pay USD 11.15m, receive EUR 10.0m.
2. **Compute the CIP-fair forward from discount factors:**
   $$F^{\star}(0,1)=X(0)\frac{P_{\mathrm{EUR}}(0,1)}{P_{\mathrm{USD}}(0,1)}=1.10\times\frac{0.99}{0.98}=1.11122449\ \text{USD/EUR}.$$
3. **Compute PV of the far-leg forward exchange (USD PV):**
   $$PV_0 = X(0)\,P_{\mathrm{EUR}}(0,1)\,N_f - F_{\mathrm{mkt}}(0,1)\,P_{\mathrm{USD}}(0,1)\,N_f.$$
   Numerically, per EUR: $1.10\times 0.99 - 1.1150\times 0.98 = -0.0037$ USD/EUR, so $PV_0\approx-\mathrm{USD} 37{,}000$.
4. **Compute FX delta (hold curves fixed):**
   $$\frac{\partial PV_0}{\partial X(0)}=P_{\mathrm{EUR}}(0,1)\,N_f \approx 0.99\times \mathrm{EUR}\,10{,}000{,}000 = \mathrm{EUR}\,9.9\text{m}.$$

**Cashflows**

| Date | Cashflow (USD) | Cashflow (EUR) | Explanation |
|---|---:|---:|---|
| 2026-02-19 | +11,000,000 | -10,000,000 | Near-leg notional exchange at spot |
| 2027-02-19 | -11,150,000 | +10,000,000 | Far-leg notional exchange at forward |

**P&L / Risk interpretation**
- If $F_{\mathrm{mkt}} \gt F^{\star}$, the USD receiver is paying “too many USD per EUR” on the far leg relative to the discount-factor parity, so the trade is negative PV (here $\approx -\mathrm{USD} 37k$).
- The FX delta in EUR units tells you the first-order PV impact (in USD) of moving the spot $X$: $\Delta PV \approx (\partial PV/\partial X)\,\Delta X$.
- In real books, $P_{\mathrm{\mathrm{USD}}},P_{\mathrm{EUR}}$ and the effective $F$ used in PV may depend on collateral and curve choices (see Section 30.4.5).

**Sanity checks**
- Units: $F$ and $X$ are USD/EUR; $P$ is unitless; PV is USD.
- Sign: receiving EUR/pay USD at maturity is long EUR forward; PV decreases when the forward $K$ increases.
- Quick parity check: $F^{\star}/X = P_{\mathrm{EUR}}/P_{\mathrm{\mathrm{USD}}} = 0.99/0.98$.

### 30.5.2 Example B: FX Swap-Implied Domestic Rate

Using the implied-rate identity from Section 30.2.2, suppose the EUR simple rate for one year is $r_f = 2.0\%$ (toy) and $F/X = 1.1150/1.10$. Then the implied USD simple rate is:

$$r_d=\frac{F}{X}(1+r_f)-1=\frac{1.1150}{1.10}\times 1.02 - 1 = 3.3818\%.$$

If the directly observed USD funding rate for the same maturity/day count differs materially from $r_d$, the difference shows up as an FX swap “basis” (after accounting for conventions and bid/ask).

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
| Notionals | $N_d = \mathrm{USD} 100{,}000{,}000$, $N_f = N_d / X(0) = \mathrm{EUR}\,90{,}909{,}090.91$ |

**Step 1: Compute forward floating rates**

Using $L = \frac{1}{\tau}\left(\frac{P^{(L)}(t_i)}{P^{(L)}(t_{i+1})} - 1\right)$:

*USD forwards:*
$$L_d(0; 0, 0.5) = \frac{1}{0.5}\left(\frac{1}{0.9890} - 1\right) = 2.2245\%$$
$$L_d(0; 0.5, 1.0) = \frac{1}{0.5}\left(\frac{0.9890}{0.9775} - 1\right) = 2.3529\%$$

*EUR forwards:*
$$L_f(0; 0, 0.5) = \frac{1}{0.5}\left(\frac{1}{0.9940} - 1\right) = 1.2072\%$$
$$L_f(0; 0.5, 1.0) = \frac{1}{0.5}\left(\frac{0.9940}{0.9880} - 1\right) = 1.2146\%$$

**Step 2: Cashflow schedule (positive = receive; receive USD float, pay EUR float + basis $b$)**

| Date | USD cashflow | EUR cashflow |
|------|-------------:|-------------:|
| 0 | $-\mathrm{USD} 100{,}000{,}000$ | $+\mathrm{EUR}\,90{,}909{,}090.91$ |
| 0.5 | $+\mathrm{USD} 1{,}112{,}235$ | $-\mathrm{EUR}\,90.91\text{m} \times (1.2072\% + b) \times 0.5$ |
| 1.0 | $+\mathrm{USD} 1{,}176{,}471$ | $-\mathrm{EUR}\,90.91\text{m} \times (1.2146\% + b) \times 0.5$ |
| 1.0 | $+\mathrm{USD} 100{,}000{,}000$ | $-\mathrm{EUR}\,90{,}909{,}090.91$ |

### 30.5.4 Example D: Solving for Par Basis Spread

We find $b_{\text{par}}$ such that the swap has zero PV at inception.

**Step 1: PV of USD leg**

- PV(coupon 0.5): $\mathrm{USD} 1{,}112{,}235 \times 0.99 = \mathrm{USD} 1{,}101{,}112$
- PV(coupon 1.0): $\mathrm{USD} 1{,}176{,}471 \times 0.98 = \mathrm{USD} 1{,}152{,}941$
- PV(principal): $\mathrm{USD} 100{,}000{,}000 \times 0.98 = \mathrm{USD} 98{,}000{,}000$

Total: $PV_d = \mathrm{USD} 100{,}254{,}053$

**Step 2: PV of EUR leg without basis**

Coupon amounts (no basis):
- At 0.5: $90{,}909{,}091 \times 1.2072\% \times 0.5 = \mathrm{EUR}\,548{,}747$
- At 1.0: $90{,}909{,}091 \times 1.2146\% \times 0.5 = \mathrm{EUR}\,552{,}080$

Discounted PV:
- PV(coupon 0.5): $\mathrm{EUR}\,548{,}747 \times 0.9950 = \mathrm{EUR}\,546{,}003$
- PV(coupon 1.0): $\mathrm{EUR}\,552{,}080 \times 0.9900 = \mathrm{EUR}\,546{,}559$
- PV(principal): $\mathrm{EUR}\,90{,}909{,}091 \times 0.9900 = \mathrm{EUR}\,90{,}000{,}000$

Total (no basis): $PV_{f,0} = \mathrm{EUR}\,91{,}092{,}562$

Converted to USD: $X(0) \cdot PV_{f,0} = 1.10 \times \mathrm{EUR}\,91{,}092{,}562 = \mathrm{USD} 100{,}201{,}818$

**Step 3: Basis PV term**

Basis adds $b \cdot \tau_i \cdot N_f$ at each payment. The PV of basis coupons:
$$PV_{\text{basis}}^{(f)} = N_f \cdot b \cdot \sum_{k=1}^{2} \tau_k P_f(0, t_k) = N_f \cdot b \cdot (0.5 \times 0.9950 + 0.5 \times 0.9900) = N_f \cdot b \times 0.9925$$

The "basis annuity": $A_f = \mathrm{EUR}\,90{,}909{,}091 \times 0.9925 = \mathrm{EUR}\,90{,}227{,}273$

**Step 4: Par condition**

$$0 = PV_d - X(0)(PV_{f,0} + A_f \cdot b_{\text{par}})$$

Solving:
$$b_{\text{par}} = \frac{PV_d / X(0) - PV_{f,0}}{A_f} = \frac{\mathrm{\mathrm{\mathrm{USD}}}\,100{,}254{,}053 / 1.10 - \mathrm{EUR}\,91{,}092{,}562}{\mathrm{EUR}\,90{,}227{,}273}$$

$$b_{\text{par}} = \frac{\mathrm{EUR}\,91{,}140{,}049 - \mathrm{EUR}\,91{,}092{,}562}{\mathrm{EUR}\,90{,}227{,}273} = \frac{\mathrm{EUR}\,47{,}487}{\mathrm{EUR}\,90{,}227{,}273} = 0.0526\% \approx 5.26 \text{ bp}$$

**Takeaway:** The par basis depends on spot FX, both discount curves, and both projection curves.

### 30.5.5 Example E: 5-Year EUR/USD Cross-Currency Swap with Basis

This example values a more realistic 5-year swap using the same framework.

**Inputs:**

| Parameter | Value |
|-----------|-------|
| Spot | $X(0) = 1.08$ USD/EUR |
| Notionals | USD 100m USD, €92.59m EUR |
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
| 1 | 4.00% | USD 4,000,000 | 0.9600 | USD 3,840,000 |
| 2 | 4.20% | USD 4,200,000 | 0.9180 | USD 3,855,600 |
| 3 | 4.30% | USD 4,300,000 | 0.8750 | USD 3,762,500 |
| 4 | 4.40% | USD 4,400,000 | 0.8320 | USD 3,660,800 |
| 5 | 4.50% | USD 4,500,000 | 0.7900 | USD 3,555,000 |
| 5 | Principal | USD 100,000,000 | 0.7900 | USD 79,000,000 |

**Total USD PV:** USD 97,673,900

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

$$PV^{(\mathrm{\mathrm{\mathrm{USD}}}\,)} = PV_d - X(0) \cdot PV_f = \mathrm{\mathrm{\mathrm{USD}}}\,97{,}673{,}900 - 1.08 \times \mathrm{EUR}\,93{,}127{,}500$$

$$PV^{(\mathrm{\mathrm{\mathrm{USD}}}\,)} = \mathrm{\mathrm{\mathrm{USD}}}\,97{,}673{,}900 - \mathrm{\mathrm{\mathrm{USD}}}\,100{,}577{,}700 = -\mathrm{\mathrm{\mathrm{USD}}}\,2{,}903{,}800$$

**Interpretation:** At 25bp basis, this swap is off-market (negative PV for the USD receiver). The par basis would be lower than 25bp given these curves.

**Step 4: Compute par basis**

Using the basis annuity approach:
$$A_f = \mathrm{EUR}\,92{,}592{,}593 \times (0.9750 + 0.9480 + 0.9200 + 0.8910 + 0.8620) = \mathrm{EUR}\,92{,}592{,}593 \times 4.596 = \mathrm{EUR}\,425{,}555{,}556$$

PV of EUR leg without basis ≈ €91,048,519 (removing the 25bp from each coupon)

$$b_{\text{par}} = \frac{\mathrm{\mathrm{\mathrm{USD}}}\,97{,}673{,}900/1.08 - \mathrm{EUR}\,91{,}048{,}519}{\mathrm{EUR}\,425{,}555{,}556} = \frac{\mathrm{EUR}\,90{,}438{,}796 - \mathrm{EUR}\,91{,}048{,}519}{\mathrm{EUR}\,425{,}555{,}556}$$

$$b_{\text{par}} \approx -14.3 \text{ bp}$$

The negative par basis indicates EUR funding is cheaper than USD funding in this curve environment—the EUR payer would receive a spread rather than pay one.

### 30.5.6 Example F: The Box Trade (Basis Extraction)

The “box” is a common way to **separate** an XCCY position into (i) single-currency curve risk in each currency and (ii) cross-currency basis risk. Conceptually, you start with an XCCY basis swap and add single-currency IRS so that the floating-index exposures largely cancel.

**Step 1: Start with an XCCY basis swap (EUR leg carries basis)**
- Receive USD floating on notional $N_{\mathrm{\mathrm{USD}}}$
- Pay EUR floating $+\ b$ on notional $N_{\mathrm{EUR}} = N_{\mathrm{\mathrm{USD}}}/X(0)$
- Vanilla form: exchange notionals at inception and maturity

**Step 2: Add single-currency IRS to offset the floating legs**
- EUR IRS: receive EUR floating, pay EUR fixed on $N_{\mathrm{EUR}}$
- USD IRS: pay USD floating, receive USD fixed on $N_{\mathrm{\mathrm{USD}}}$

After these hedges, the EUR and USD floating coupons are largely removed. What remains is:
- The **basis coupons** (paying or receiving $b$ on the basis leg)
- Two **fixed legs** (one in each currency) whose PV depends on their respective discount curves
- **FX delta** because any foreign-currency PV (including notional exchanges and basis coupons) is converted at spot

**Basis-only P&L approximation (hold curves and spot fixed):**
$\Delta PV \approx \frac{\partial PV}{\partial b}\,\Delta b$. For a basis payer, $\frac{\partial PV}{\partial b} \lt 0$, so a 10 bp tightening (basis down 10 bp) produces a positive P&L of approximately $10 \times$ basis DV01.

> **Desk Reality: What the “box” is (and isn’t)**
>
> The box is best thought of as a **risk decomposition tool**, not a guarantee of “pure basis”:
> - Hedge effectiveness depends on matching schedules, day counts, and collateral/discounting assumptions.
> - If your system “bumps and rebuilds” curves when $b$ moves, some curve risk can reappear through the rebuild.
> - FX notional exchanges can leave meaningful FX delta unless separately hedged.

---

## 30.6 Risk Decomposition

Risk numbers depend on (i) what is bumped and (ii) what is held fixed. Throughout this chapter we use:

- **1 bp** $=\ 10^{-4}$.
- PV is reported in the **domestic currency** $d$ and **positive PV means an asset**.
- **Discount-curve DV01 (per currency):** a **parallel down bump** of the continuously-compounded zero-rate curve used for **discounting** in that currency, with
  $$DV01 := PV(\text{rates down }1\text{ bp}) - PV(\text{base}) \qquad [\text{domestic currency per bp}].$$
  Unless stated otherwise, hold spot $X(0)$, FX forwards, and the other currency’s curves fixed (a “hold-fixed” decomposition).
- **Basis DV01:** bump the quoted basis spread $b$ **down** by 1 bp on the basis coupons only, holding curves fixed; report $DV01_b := PV(b\downarrow 1\text{ bp}) - PV(\text{base})$.
- **FX delta:** bump spot $X(0)$ (domestic per 1 foreign) holding curves fixed; $\partial PV/\partial X$ naturally has units of **foreign currency**.

### 30.6.1 FX Delta

For a position valued as $PV^{(d)} = PV_d - X(0) \cdot PV_f^{(f)}$, holding all curves and the foreign PV fixed:

$$\boxed{\frac{\partial PV^{(d)}}{\partial X(0)} = -PV_f^{(f)}}$$

**Interpretation:** If you pay the foreign leg, you are “short foreign PV” in domestic terms. When the foreign currency strengthens ($X$ rises for a domestic/foreign quote), the foreign leg becomes more expensive, reducing your PV.

**Units:** $\partial PV^{(d)}/\partial X$ has units of foreign currency (here: EUR) because $X$ is USD/EUR.

**From Example D:** FX delta $\approx -\mathrm{EUR}\,91.14\text{m}$. For $X(0)=1.10$, a +1% spot shock is $\Delta X \approx 0.011$, so
$$\Delta PV \approx -\mathrm{EUR}\,91{,}140{,}049 \times 0.011 = -\mathrm{\mathrm{\mathrm{USD}}}\,1{,}002{,}540$$

### 30.6.2 Domestic and Foreign DV01 (Discount Curves)

For a receive-USD-float / pay-EUR-float-plus-basis position, there are two discount curves to bump:

- **Domestic DV01:** bump the domestic discount curve down 1 bp, holding foreign curves and FX fixed.
- **Foreign DV01:** bump the foreign discount curve down 1 bp, holding domestic curves and FX fixed.

**From Example D calculations (hold-fixed):**
- A +1 bp *increase* in USD discount rates reduces PV by about $\mathrm{USD} 9{,}771$, so $DV01_{\mathrm{\mathrm{USD}}} \approx +\mathrm{USD} 9{,}771/\text{bp}$.
- A +1 bp *increase* in EUR discount rates increases PV by about $\mathrm{USD} 9{,}893$, so $DV01_{\mathrm{EUR}} \approx -\mathrm{USD} 9{,}893/\text{bp}$.

**Note on bump schemes:** In an arbitrage-consistent setup, bumping one curve may require re-implying FX forwards or other curves (“bump-and-rebuild”). Different “hold fixed” choices produce different DV01s; document the bump object and what is held fixed.

### 30.6.3 Basis DV01

The basis spread $b$ enters additively in the foreign leg coupons. From the XCCY-PV formula:

$$\boxed{\frac{\partial PV}{\partial b} = -X(0) \cdot N_f \cdot \sum_i \tau_i P_f(0, t_{i+1})}$$

This is negative for the basis payer: when the basis you pay widens, your PV falls. With the chapter DV01 convention (basis **down** 1 bp),
$$DV01_b \approx -\frac{\partial PV}{\partial b}\times 10^{-4}.$$

**From Example D:**
$$DV01_b = 1.10 \times \mathrm{EUR}\,90{,}909{,}091 \times 0.9925 / 10{,}000 \approx +\mathrm{\mathrm{\mathrm{USD}}}\,9{,}925 \text{ per bp}$$

A +5bp widening (basis up) costs approximately $5 \times \mathrm{USD} 9{,}925 \approx \mathrm{USD} 49{,}625$ for the basis payer.

### 30.6.4 Risk Summary Table

For a receive-USD-float / pay-EUR-float-plus-basis position on USD 100m:

| Risk Factor | Sensitivity | Sign Intuition |
|-------------|-------------|----------------|
| FX delta | -€91.14m | Short EUR: EUR strength hurts |
| USD DV01 (rates down) | +USD 9,771/bp | Long USD cashflows: lower rates help |
| EUR DV01 (rates down) | -USD 9,893/bp | Short EUR cashflows: lower rates hurt |
| Basis DV01 (basis down) | +USD 9,925/bp | Paying basis: tighter basis helps |

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

**Scenario:** A German corporation has issued USD 200m of USD-denominated bonds to access the deeper U.S. market. The USD coupons create FX exposure for a company whose revenues are in EUR.

**Solution:** Enter a cross-currency swap:
- Pay USD floating (to offset USD bond coupons, possibly with a USD fixed-to-float swap first)
- Receive EUR floating + basis

**Effect:**
- USD debt service transformed into EUR-linked payments
- Natural hedge against EUR revenues
- The basis spread determines the "all-in" EUR funding cost

### 30.7.3 Transforming the Currency of Funding

**Scenario:** You can borrow cheaply in one currency but need to fund assets in another currency for months or years.

**Solution:**
1. Borrow in the “cheap” currency.
2. Use an FX swap (short tenor) or an XCCY basis swap (multi-period) to obtain the funding currency.

**Economics (in words):**
- The swap converts one currency’s floating funding into another’s.
- The quoted basis $b$ (and which leg it applies to) determines the incremental spread you pay/receive on top of the target-currency floating rate.
- Always translate the quote into cashflows and compute PV/risk under stated curve and collateral assumptions before interpreting “cheap” vs “expensive”.

### 30.7.4 The Box Trade: Risk Decomposition

The “box” uses an XCCY basis swap plus single-currency IRS to largely remove the floating-index exposures, leaving a package whose dominant sensitivity is to the cross-currency basis (and still to FX). See Example F (Section 30.5.6).

Key points:
- It is not automatically “pure basis”: FX delta, schedule mismatches, and curve rebuild effects can matter.
- Hedge sizing depends on the bump scheme used for DV01 and basis DV01.

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

1. **Mixing projection and discount curves:** The projection curve generates forward rates; the discount curve PVs cashflows. Do not use one for both.

2. **Inconsistent FX conversion:** If you convert foreign cashflows using forwards, ensure $F(0,T) = X(0) P_f(0,T)/P_d(0,T)$ holds—or acknowledge you are accepting a "model inconsistency" for risk decomposition.

3. **Treating basis as "just a spread":** The basis embeds curve dependencies. Changing basis quotes requires consistent curve re-bootstrapping, not just adding a spread.

4. **Confusing FX swaps and XCCY swaps:** FX swaps are often used for shorter tenors; multi-period currency transformation is often done with XCCY swaps. Always write down the cashflows you are pricing.

### 30.8.3 Implementation Pitfalls

1. **Calendar misalignment:** Payment dates may not align across currencies. A robust date engine handling multiple holiday calendars is essential.

2. **Interpolation artifacts:** If implied forward FX is computed from interpolated discount factors, ensure the FX curve is smooth and consistent with the basis curve.

3. **Bump-and-rebuild vs. hold-fixed:** When computing DV01/basis DV01, decide whether to hold other curves/FX fixed or to re-bootstrap consistently. Document the choice.

### 30.8.4 Value Dates and Operational Checks

Even when the pricing math is correct, many breaks come from inconsistent **dates**:

- **Trade date:** when the deal is agreed.
- **Spot/value date:** when the near-leg notional exchange settles (often T+2 in many major pairs, but exceptions exist).
- **Forward value dates:** the settlement dates for the far leg (FX swap) or for each coupon date (XCCY swap).

**Practical checks:**
1. Compute accrual fractions $\tau_i$ from the correct value dates (not from trade date).
2. Build payment schedules per leg using each currency’s calendar, then reconcile mismatches explicitly.
3. For same-day payments in two currencies, confirm whether payment netting applies (and under what documentation).

> **Desk Reality: “Correct” PV, wrong cash**
>
> Reconciliation breaks often trace to operational mismatches rather than valuation formulas. When investigating a break, check:
> - Quote direction / forward points conversion
> - Value dates and calendars (near leg vs far leg)
> - Whether notionals are exchanged at inception/maturity as modeled
> - Fixing conventions and cutoffs

---

## Summary

1. **Quote → cashflows → PV:** FX forwards/swaps translate spot/points and value dates into explicit notional exchanges and PV.
2. **CIP anchor (DF form):** $F(0,T)=X(0)\,P_f(0,T)/P_d(0,T)$ is the internal-consistency check tying spot, forwards, and discount factors.
3. **FX forward PV:** receiving foreign and paying $K$ domestic/foreign at $T$ has $PV^{(d)} = X(0)P_f(0,T) - K P_d(0,T)$ per unit of foreign notional.
4. **FX swaps imply funding:** a near-leg spot exchange plus a far-leg forward reversal implies a short-dated funding rate via $1+r_d\tau = (F/X)(1+r_f\tau)$ once conventions are aligned.
5. **XCCY basis swaps price as “domestic PV − spot × foreign PV”:** the basis $b$ enters additively in one leg’s floating coupons and is solved as a par spread (or paired with an upfront).
6. **Multi-curve plumbing matters:** projection curves generate forwards; discount curves PV cashflows; basis depends on both (and on collateral assumptions).
7. **Risk must state the bump scheme:** FX delta, domestic/foreign discount-curve DV01s, and basis DV01 depend on what is bumped and what is held fixed vs rebuilt.
8. **Common breaks:** quote direction/points sign, which leg carries basis, value dates/calendars, and inconsistent curve/FX inputs.

---

## Key Concepts

| Concept | Definition | Why It Matters |
|---|---|---|
| Spot FX $X(0)$ | Quote in domestic per 1 foreign (e.g., USD/EUR) | Fixes PV units and hedge ratios; wrong direction creates large errors |
| Forward FX $F(0,T)$ | Future exchange rate for value date $T$ | Determines far-leg cashflows for FX forwards/swaps |
| Forward points | Quoted points added to spot to obtain $F$ | Separates spot risk from carry/rate-differential risk in workflow |
| Covered interest parity (CIP) | $F(0,T)=X(0)\,P_f(0,T)/P_d(0,T)$ under consistent inputs | Consistency check across curves and FX forwards |
| FX forward PV | $PV^{(d)} = X P_f - K P_d$ per unit foreign (receive foreign, pay $K$ domestic/foreign) | Converts a quote into PV and FX delta |
| FX swap | Near-leg spot exchange + far-leg forward reversal | Desk funding instrument; implies a short-dated rate once conventions align |
| XCCY basis swap | Exchange floating legs in two currencies; one leg has spread $b$ | Prices currency-transformed funding; basis risk is a primary driver |
| Par basis $b^{\text{par}}$ | Basis that makes PV = 0 with zero upfront (given curves/CSA) | Connects market quotes to valuation and curve building |
| Projection vs discount curves | Projection generates forwards; discount PVs cashflows | Mixing them is a common implementation error |
| Collateral currency | CSA remuneration rate drives discounting under idealized assumptions | Changes PV and reported risk across systems |
| FX delta | $\partial PV^{(d)}/\partial X$ under a stated hold-fixed scheme | Drives hedging with spot/forward instruments |
| DV01 / basis DV01 | $DV01 := PV(\text{rates down }1bp)-PV(\text{base})$ (and similarly for $b$) | Risk reporting must state bump object, size, and what’s held fixed |

---

## Notation

| Symbol | Meaning | Units / Convention |
|---|---|---|
| $d,f$ | domestic / foreign currency | PV is reported in $d$ |
| $X(0)$ | spot FX (domestic per 1 foreign) | e.g., USD/EUR |
| $F(0,T)$ | forward FX for value date $T$ | domestic per 1 foreign |
| $P_d(0,T)$ | domestic discount factor | unitless; $P(T,T)=1$ |
| $P_f(0,T)$ | foreign discount factor | unitless |
| $P_c^{(L)}(0,T)$ | projection-curve DF for index $L$ in currency $c$ | used to generate forwards |
| $L_c(t_i,t_{i+1})$ | forward rate over $[t_i,t_{i+1}]$ | decimal per year |
| $\tau_i$ | accrual fraction for $[t_i,t_{i+1}]$ | year fraction |
| $N_d, N_f$ | notionals in domestic/foreign currency | currency units |
| $b$ | quoted basis spread (applied to stated leg) | decimal per year; 1 bp $=10^{-4}$ |
| $PV^{(d)}$ | present value in domestic currency | domestic currency |
| $DV01$ | $PV(\text{rates down }1bp)-PV(\text{base})$ | domestic currency per 1 bp; bump object must be stated |
| $DV01_b$ | $PV(b\downarrow 1bp)-PV(\text{base})$ | domestic currency per 1 bp |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | Write covered interest parity (DF form) | $F(0,T)=X(0)\,P_f(0,T)/P_d(0,T)$ |
| 2 | PV of an FX forward (receive 1 foreign, pay $K$ domestic at $T$) | $PV^{(d)}=X(0)P_f(0,T)-K P_d(0,T)$ |
| 3 | What is an FX swap? | A near-leg spot exchange plus a far-leg forward reversal |
| 4 | Why quote forward points instead of outright forwards? | To quote the carry / rate-differential component separately from the spot level |
| 5 | What is “par basis”? | The $b$ that makes PV $=0$ with zero upfront, given curves/CSA assumptions |
| 6 | High-level XCCY valuation decomposition | $PV^{(d)} = PV_d^{(d)} - X(0)\,PV_f^{(f)}$ with clearly defined legs |
| 7 | Projection vs discount curve: which does what? | Projection generates forwards; discount PVs cashflows |
| 8 | Spot-discount vs forward-discount: when do they agree? | When inputs are CIP-consistent (and you are not deliberately holding linked objects fixed while bumping others) |
| 9 | FX delta for $PV^{(d)} = PV_d - X\,PV_f$ (hold-fixed) | $\partial PV/\partial X = -PV_f$ (units: foreign currency) |
| 10 | DV01 sign convention used in this book | $DV01 := PV(\text{rates down }1\text{bp}) - PV(\text{base})$ |
| 11 | What must a DV01 report specify? | Bump object (which curve/spread), bump size, and what’s held fixed vs rebuilt |
| 12 | Basis DV01 for a basis payer (hold-fixed) | Positive for “basis down 1bp”; PV decreases when basis widens |
| 13 | One date-related break for FX swaps | Using trade date instead of value dates for $\tau$ and discounting |
| 14 | One confirmation-related break for XCCY | Whether notionals are exchanged and which leg carries basis |
| 15 | Why can collateral currency change PV? | It changes discounting/FX conversion assumptions for collateralized cashflows |
| 16 | What does the “box” do conceptually? | Uses IRS hedges to reduce single-currency floating risk, leaving mainly basis (and FX) |

---

## Mini Problem Set

1. (Compute) $X(0)=1.25$, $P_d(0,1)=0.96$, $P_f(0,1)=0.98$. Compute the no-arbitrage forward $F(0,1)$.
2. (Compute) You receive $\mathrm{EUR}\,1{,}000{,}000$ and pay $\mathrm{USD} 1{,}300{,}000$ at $T=1$. Given $X(0)=1.28$ USD/EUR, $P_d(0,1)=0.97$, $P_f(0,1)=0.98$, compute PV in USD.
3. (Compute) An FX swap has $F/X = 1.006$, foreign simple rate $r_f=2.0\%$, and $\tau=0.5$. Compute the implied domestic simple rate $r_d$.
4. (Compute) A basis payer has $X(0)=1.10$, $N_f=\mathrm{EUR}\,50\text{m}$, semiannual accruals $\tau_1=\tau_2=0.5$, and foreign discount factors $P_f(0,0.5)=0.995$, $P_f(0,1.0)=0.990$. Compute $DV01_b$ in USD (basis down 1bp, hold-fixed).
5. (Concept) Give one reason the two FX conversion approaches in Section 30.4.4 can disagree even if each is coded correctly.
6. (Concept) A report shows “USD DV01 = USD 12k/bp” for an XCCY book. List two questions you would ask to interpret the number.
7. (Desk) You see a persistent PV break between two systems for the same XCCY trade. List three common root causes to check first.
8. (Desk) In one sentence, explain why “basis is just a spread” is an unsafe implementation mindset.
9. (Compute) Using Example D’s basis DV01 magnitude $\approx \mathrm{USD} 9{,}925/\text{bp}$ for the basis payer, estimate the PV impact of a +7 bp basis widening (hold-fixed).
10. (Concept) If you change quote direction from USD/EUR to EUR/USD, what must you do to keep FX delta and hedges consistent?

### Solution Sketches (Selected)

1. $F = X\,P_f/P_d = 1.25\times 0.98/0.96 = 1.2760$ USD per foreign.
2. Use $PV = X P_f N_f - K P_d N_f$ with $N_f=\mathrm{EUR}\,1\text{m}$, $K=1.30$: $1.28\times 0.98\times 1{,}000{,}000 - 1.30\times 0.97\times 1{,}000{,}000 = -\mathrm{USD} 6{,}600$.
3. $1+r_d\tau=(F/X)(1+r_f\tau)\Rightarrow r_d = \big((F/X)(1+r_f\tau)-1\big)/\tau$. Here $r_d=((1.006)(1+0.02\times 0.5)-1)/0.5=3.212\%$.
4. $DV01_b = X N_f\sum_i \tau_i P_f /10{,}000$. Here $\sum \tau P_f=0.5(0.995)+0.5(0.990)=0.9925$, so $DV01_b=1.10\times 50{,}000{,}000\times 0.9925/10{,}000\approx \mathrm{USD} 5{,}459/\text{bp}$.
5. If you “bump one input” (e.g., a curve) but hold other objects fixed (e.g., FX forwards) that are linked by CIP in the base calibration, the two approaches will no longer be consistent by construction.
7. Common checks: (i) CSA/collateral assumptions and discount curves, (ii) value dates/calendars and $\tau$, (iii) quote direction/points conversion and which leg carries basis.
9. A +7 bp widening means $b$ increases by 7 bp, so PV decreases by about $7\times \mathrm{USD} 9{,}925 \approx \mathrm{USD} 69{,}475$ for the basis payer (hold-fixed).

---

## References

- (Andersen and Piterbarg, *Interest Rate Modeling*, “Notations and FX Forwards”; “Cross-Currency Basis Swaps”)
- (Neftci, *Principles of Financial Engineering*, “Swap Engineering in FX Markets”; settlement/value date conventions; forward points discussion)
- (Brigo, Morini, Pallavicini, *Counterparty Credit Risk, Collateral and Funding*, “Pricing with Two Currencies: Foundations”)
- (Hull, *Options, Futures, and Other Derivatives*, currency swaps / FX forwards chapters)
- (Taleb, *Dynamic Hedging*, “Foreign Currency Forwards” and forward points discussion)
