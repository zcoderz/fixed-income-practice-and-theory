# Chapter 29: FX Spot and Forwards — Pricing via Interest Differentials (Forward Points, Carry, Hedging)

---

## Fact Classification

### (A) Verified Facts (Source-Backed)
- Interest rate parity / CIP formula: $F_0 = S_0 e^{(r - r_f)T}$ (Hull Ch 5)
- Foreign currency treated as asset providing yield at foreign risk-free rate (Hull Ch 5)
- Forward exchange rate from numeraire ratio: $W(t) = \frac{P_X(t,T)}{P_Y(t,T)} S(t)$ (Hull)
- Multi-currency valuation uses domestic value of foreign ZCB: $\tilde{P}_D(t,T) = X(t) P_F(t,T)$ (Andersen & Piterbarg Vol 1)
- FX forward market rarely liquid beyond ~1 year; longer horizons use cross-currency basis swaps (Andersen & Piterbarg Vol 1)
- One-period cross-currency basis swap identical to FX forward contract (Andersen & Piterbarg Vol 1)
- Forward delta for asset with yield $q$: $\Delta = e^{-qT}$ (Hull Ch 5)

### (B) Reasoned Inference (Derived from A)
- DF-form CIP: $F(t,T) = S_t \frac{P_F(t,T)}{P_D(t,T)}$ — derived from Hull's numeraire ratio
- PV formula: $V_t^{(D)} = N_F(S_t P_F(t,T) - K P_D(t,T))$ — derived from discounting each currency leg
- Quote inversion inverts the DF ratio: $F^{F/D} = S^{F/D} \frac{P_D(t,T)}{P_F(t,T)}$
- Forward points sign depends on rate differential and quote direction

### (C) Speculation (Clearly Labeled; Minimal)
- Forward points scaling conventions (pips/points) — **not sourced**
- FX spot settlement conventions (T+2, T+1) — **not sourced**
- NDF settlement mechanics — **not sourced**
- Triangular arbitrage mechanics — **not explicitly sourced**

---

## Conventions & Notation

| Symbol | Meaning | Units |
|--------|---------|-------|
| $D$, $F$ | Domestic and foreign currencies | — |
| $S_t^{D/F}$ | Spot FX at time $t$ | $D$ per $F$ |
| $S_t^{F/D} = 1/S_t^{D/F}$ | Inverse spot quote | $F$ per $D$ |
| $F(t,T)$ | Forward exchange rate for maturity $T$ | $D$ per $F$ |
| $K$ | Delivery price (strike) of FX forward | $D$ per $F$ |
| $P_D(t,T)$ | Domestic discount factor (ZCB price) | dimensionless |
| $P_F(t,T)$ | Foreign discount factor (ZCB price) | dimensionless |
| $N_F$ | Foreign currency notional | $F$ |
| $T$ | Maturity (year-fraction) | years |

**Default quote direction:** $S_t^{D/F}$ (domestic per 1 foreign) throughout this chapter. This matches the "domestic per foreign" convention used in the multi-currency setup in *Interest Rate Modeling*.

**Long FX forward (buy foreign):** Receive $N_F$ foreign at $T$, pay $K N_F$ domestic at $T$.

**Key conventions:**
- Use CIP / interest rate parity as the no-arbitrage pricing backbone
- Valuation via discount factors $P_D(t,T)$, $P_F(t,T)$ whenever possible (avoids compounding ambiguity)
- Foreign PV to domestic PV conversion: $X(t) P_F(t,T)$ in domestic units

---

## Core Concepts

### 1) Spot FX and Quote Direction

**Formal Definition:**
A spot FX rate $S_t^{D/F}$ is the price of 1 unit of foreign currency in units of domestic currency (domestic per foreign).

Market quoting can use either direction; Hull notes that in many major pairs the spot/forward exchange rate is "normally quoted as the number of units of the currency that are equivalent to one U.S. dollar," with some exceptions for certain currencies.

**Intuition:**
$S_t^{D/F}$ answers: "How many dollars do I pay for 1 euro?" (if $D =$ USD, $F =$ EUR).

Inverting the quote changes the numerical value and flips which rate is "up" vs "down."

**Trading / Risk Practice:**
Risk systems must store both the pair and the quote direction. Mis-storing the inverse is a common source of sign errors in P&L and hedges.

---

### 2) FX Forward Contract and Forward Exchange Rate

**Formal Definition:**
A forward contract is an agreement made at time $t$ to exchange assets at a future time $T$. In fixed income terminology, the forward's value is typically set to zero at initiation (delivery price chosen so $V_t = 0$).

An FX forward specifically exchanges two currencies at $T$: pay $K N_F$ domestic and receive $N_F$ foreign (deliverable forward).

**Intuition:**
The forward locks in an exchange rate for the future, removing FX uncertainty for a known foreign-currency cashflow.

**Trading / Risk Practice:**
Hull illustrates hedging foreign-currency exposure: if a firm will receive a foreign currency amount in the future, it can hedge by entering a forward to sell that foreign amount for domestic currency at a fixed rate, thereby eliminating FX risk (in the idealized setting).

---

### 3) Covered Interest Parity (CIP) / Interest Rate Parity

**Formal Definition:**
CIP is the no-arbitrage condition that links $S_t$, $F(t,T)$, and domestic/foreign interest rates (or discount factors). Hull presents this as "interest rate parity" for currency forwards:

$$\boxed{F_0 = S_0 e^{(r - r_f)T}}$$

where $r$ is the domestic risk-free rate and $r_f$ is the foreign risk-free rate (continuous compounding).

**Intuition:**
You can synthesize a forward FX conversion by:
1. Converting at spot today
2. Investing in one currency
3. Borrowing in the other

No-arbitrage forces the forward to equal the rate that prevents "free money" from covered borrowing/lending loops.

**Trading / Risk Practice:**
- Traders often describe forwards as "spot + carry," where carry is driven by the interest differential
- *Interest Rate Modeling* emphasizes that FX forwards and cross-currency instruments impose arbitrage constraints across currencies; failure to fit quoted instruments can create inconsistencies (previewed later as cross-currency basis)

---

### 4) Forward Points

**Formal Definition (Generic):**
$$\text{Forward points} = F(t,T) - S_t$$

**I'm not sure.** The provided sources do not explicitly define "forward points" or specify pip/point scaling conventions for different currency pairs. To be precise for a desk, we would need: (i) currency pair, (ii) market quoting convention for points/pips, and (iii) settlement/spot-date conventions.

**Intuition:**
Forward points reflect the interest differential over the term, translated into FX units.

**Trading / Risk Practice:**
For liquid pairs, forwards may be quoted as "spot + points." Risk systems must handle points scaling carefully (pair-specific).

---

### 5) Carry and Hedging Implications

**Formal Definition:**
"Carry" is the expected P&L from holding a position assuming prices don't move, coming from financing/yield differences. In fixed-income terms, carry is the benefit/cost from "simply holding the bond" when yield/curve doesn't change.

In FX forwards, carry is primarily the interest differential embedded in $F/S$.

**Intuition:**
If domestic rates exceed foreign rates, the domestic price of foreign currency tends to be higher in the forward (in domestic-per-foreign quotes), giving positive forward points.

Reverse happens if foreign rates exceed domestic.

**Trading / Risk Practice:**
- **Currency hedging:** A domestic investor holding a foreign asset can hedge FX risk by selling the foreign currency forward (matching the foreign notional exposure) to lock the domestic value at maturity
- **Rolling a hedge:** When the underlying asset horizon exceeds available forward tenors, hedgers "roll" short-dated forwards (similar to stack-and-roll hedging for futures/forwards)

---

### 6) Link to Chapter 21 (Cross-Currency Curves): DF-Ratio Perspective and Basis Preview

**Formal Definition:**
Multi-currency valuation uses domestic and foreign discount curves $P_D(t,T)$, $P_F(t,T)$ and spot FX $X(t)$. *Interest Rate Modeling* sets this up explicitly and uses FX forwards as part of cross-currency curve construction.

In its cross-currency curve construction, the book derives relationships tying discount factors and FX forwards (e.g., equation (6.38)), and stresses that violations create arbitrage opportunities.

**Intuition:**
CIP is not just a single-maturity identity: applied across maturities, it links two entire discount curves to the term structure of FX forwards.

**Trading / Risk Practice:**
*Interest Rate Modeling* notes the interbank FX forward market is "rarely liquid beyond maturities of one year" and longer horizons use cross-currency basis swaps; a one-period basis swap is identical to an FX forward.

This chapter previews that basis/collateral/funding can alter the "effective" curve inputs (handled in Ch 21).

---

## Math and Derivations

### 2.1 Payoff of an FX Forward in Two Currencies

Consider a long FX forward (buy foreign) with foreign notional $N_F$ and delivery price $K$ (units $D/F$) maturing at $T$.

**Currency cashflows at $T$:**
- Receive: $+N_F$ units of foreign currency $F$
- Pay: $-K N_F$ units of domestic currency $D$

If you translate the payoff into domestic units at maturity using spot $S_T$:

$$\boxed{\text{Domestic-equivalent payoff at } T = N_F(S_T - K)}$$

**Interpretation:** You effectively "bought foreign at $K$" vs "could have bought at spot $S_T$."

**Unit check:** $S_T$ and $K$ are $D/F$, so $(S_T - K) N_F$ is $D$. $\checkmark$

---

### 2.2 Covered Interest Parity: Forward from Interest Differential (Rates Form)

Hull's currency-forward pricing expresses interest rate parity in continuous-compounding terms:

$$F_0 = S_0 e^{(r - r_f)T}$$

where $r$ is domestic and $r_f$ is foreign.

**Economic replication intuition (borrow–convert–invest–sell forward):**
Treat 1 unit of foreign currency as an "asset" that earns the foreign risk-free rate, analogous to a continuous dividend yield. Hull explicitly treats foreign currency this way: "the foreign currency can be considered as an asset providing a yield at a rate equal to the foreign risk-free interest rate."

**Simple-rate version (same logic, different compounding):**
If you represent funding by simple accrual over year fraction $T$:

$$F_0 = S_0 \frac{1 + R_D T}{1 + R_F T}$$

where $R_D, R_F$ are simple annualized rates.

This is the same identity expressed via discount factors (next section), provided rates are translated consistently.

**Sanity check:** If $r = r_f$, then $F_0 = S_0$. $\checkmark$

---

### 2.3 Covered Interest Parity: Forward from Ratio of Discount Factors (DF Form)

A robust way to avoid compounding ambiguity is to use discount factors.

Hull's multi-currency numeraire ratio gives a forward exchange rate directly:

$$W(t) = \frac{P_X(t,T)}{P_Y(t,T)} S(t)$$

and states $W(t)$ is the forward exchange rate (units of $Y$ per unit of $X$) for maturity $T$.

**Map to our domestic/foreign convention:**
- Let $X = F$ (foreign), $Y = D$ (domestic), and $S(t) = S_t^{D/F}$
- Then $P_X = P_F$, $P_Y = P_D$, and the forward in domestic-per-foreign units is:

$$\boxed{F(t,T) = S_t \frac{P_F(t,T)}{P_D(t,T)}}$$

*Interest Rate Modeling* uses the same structure in its multi-currency setup and cross-currency curve construction, linking domestic and foreign discount factors with FX forwards (see equation (6.38)).

**Unit check:** $P_F / P_D$ is dimensionless, so $F(t,T)$ has units $D/F$. $\checkmark$

---

### 2.4 PV of an FX Forward: Discount the Two Currency Legs Consistently

Start from cashflows:
- $+N_F$ foreign at $T$
- $-K N_F$ domestic at $T$

**Step 1: PV of the domestic leg (in domestic currency):**
$$PV_D = -K N_F \, P_D(t,T)$$

**Step 2: PV of the foreign leg, in domestic currency.**

From *Interest Rate Modeling*, the domestic value of one foreign zero-coupon bond is:
$$\tilde{P}_D(t,T) = X(t) P_F(t,T)$$

with $X(t)$ the spot FX in domestic per foreign units.

So PV of receiving $N_F$ foreign at $T$ is:
$$PV_F = +N_F \, S_t \, P_F(t,T)$$

**Therefore the domestic PV of the FX forward is:**

$$\boxed{V_t^{(D)} = N_F \left( S_t P_F(t,T) - K P_D(t,T) \right)}$$

**Connection to Hull's forward-valuation formula:** Hull gives the value of a forward on an investment asset with yield $q$:
$$f_0 = S_0 e^{-qT} - K e^{-rT}$$

For FX, set $q = r_f$ (foreign rate as yield) and interpret $e^{-rT}, e^{-r_f T}$ as discount factors under flat continuous curves.

---

### 2.5 Fair Forward and the Standard Simplification $V = P_D(F^* - K)$

Define the fair (no-arbitrage) forward:

$$F^*(t,T) = S_t \frac{P_F(t,T)}{P_D(t,T)}$$

Then:
$$S_t P_F(t,T) = F^*(t,T) P_D(t,T)$$

so:

$$\boxed{V_t^{(D)} = N_F P_D(t,T) \left( F^*(t,T) - K \right)}$$

**Sanity check:** If the contract is struck at $K = F^*$, then $V_t^{(D)} = 0$ (consistent with "fair forward has zero value at inception"). $\checkmark$

---

### 2.6 Quote Inversion: How Formulas Change When You Flip the Quote

If the market quotes the inverse:

$$S_t^{F/D} = \frac{1}{S_t^{D/F}}, \qquad F^{F/D}(t,T) = \frac{1}{F^{D/F}(t,T)}$$

From DF-form CIP:
$$F^{D/F}(t,T) = S_t^{D/F} \frac{P_F(t,T)}{P_D(t,T)}$$

Invert:
$$F^{F/D}(t,T) = \frac{1}{S_t^{D/F}} \frac{P_D(t,T)}{P_F(t,T)} = S_t^{F/D} \frac{P_D(t,T)}{P_F(t,T)}$$

**Key point:** Inverting the quote inverts the DF ratio.

---

### 2.7 Forward Points and the Interest Differential

Using DF-form CIP:

$$F(t,T) - S_t = S_t \left( \frac{P_F(t,T)}{P_D(t,T)} - 1 \right)$$

- If domestic rates are higher than foreign rates, typically $P_D(t,T) < P_F(t,T)$ for the same maturity, so $\frac{P_F}{P_D} > 1$ and $F > S$: "forward premium" in domestic-per-foreign quotes
- Reverse happens if foreign rates exceed domestic

**Caution on sign conventions:** Even for futures hedging, Hull notes conventions can differ (e.g., basis sometimes defined as spot–futures or futures–spot). Forward points conventions (sign, scaling) are similarly desk/market dependent.

---

### 2.8 Hedging a Foreign-Currency Asset Using FX Forwards

Suppose you will receive a foreign-currency payoff $A_F(T)$ (units: $F$) at time $T$.

**Unhedged domestic value at $T$:** $S_T A_F(T)$ (random due to $S_T$).

**Hedge with a short forward** to sell $N_F = A_F(T)$ foreign at delivery price $K$:
- At $T$, you deliver $A_F(T)$ foreign and receive $K A_F(T)$ domestic
- Domestic payoff becomes $K A_F(T)$, eliminating $S_T$ risk (in the idealized setting)

Hull's corporate example motivates the same idea: if a firm expects a known foreign currency amount, selling it forward locks the domestic amount and removes FX risk.

**Where interest differentials enter:** If you choose $K = F^*(t,T)$, then:
$$K = S_t \frac{P_F(t,T)}{P_D(t,T)}$$

so the locked-in domestic conversion rate differs from today's spot by the DF ratio, i.e., the interest differential.

---

## Measurement & Risk

### 3.1 Mark-to-Market (MTM) of an FX Forward

**Domestic PV:**
$$V_t^{(D)} = N_F \left( S_t P_F(t,T) - K P_D(t,T) \right) = N_F P_D(t,T) \left( F^*(t,T) - K \right)$$

**Desk interpretation:** "PV is discounted (fair forward − trade forward) × notional."

---

### 3.2 Spot FX Sensitivity ("FX Delta")

Differentiate PV w.r.t. spot $S_t$ (holding curves fixed):

$$\boxed{\frac{\partial V_t^{(D)}}{\partial S_t} = N_F P_F(t,T)}$$

Under flat continuous rates, Hull's forward-delta result for an investment asset with yield $q$ is $e^{-qT}$; for a forward FX contract, set $q = r_f$ (foreign rate). This aligns with the intuition that the delta is the PV factor for the foreign leg.

**Unit check:** $P_F$ is dimensionless; derivative units are $N_F$ (foreign) times $D/F$ change in spot → domestic currency PV change. $\checkmark$

---

### 3.3 Interest-Rate Curve Sensitivities (Domestic vs Foreign)

From:
$$V_t^{(D)} = N_F \left( S_t P_F(t,T) - K P_D(t,T) \right)$$

- Exposure to **foreign curve** is through $P_F(t,T)$ (PV of foreign leg)
- Exposure to **domestic curve** is through $P_D(t,T)$ (PV of domestic payment)

Practically, if the desk uses different collateral/funding curves (multi-curve), the choice of $P_D, P_F$ becomes critical (previewed in cross-currency curve construction).

---

### 3.4 Hedge Sizing for a Foreign Asset

If you know a future foreign cashflow amount $A_F(T)$, a simple hedge is $N_F = A_F(T)$ short forward (sell foreign forward) to lock domestic conversion.

If the foreign asset's value is stochastic in foreign currency (e.g., equity), you hedge an estimated foreign value (e.g., current foreign market value, or a target hedge ratio), and accept residual risk from foreign asset price changes.

---

### 3.5 Rolling Hedge Risk Decomposition (Conceptual)

When you repeatedly hedge short-dated forwards (a "stack and roll" style approach), realized hedged P&L decomposes into:
1. Spot move over each hedge period
2. Carry from forward points (interest differential)
3. Curve move (changes in $P_D, P_F$, hence changes in forward curve)
4. Operational frictions (bid/ask, settlement conventions)

---

## Worked Examples

> **Global note:** Numbers are illustrative. If your desk uses specific spot-date conventions (e.g., T+2, T+1) or day-count conventions for money-market rates, you must adapt $T$ (year fraction) and discount factors accordingly. **I'm not sure** about FX spot settlement conventions — the provided sources do not specify them.

---

### Example A — Quote Direction Sanity (Invert Spot; Invert Forward; Unit Checks)

**Conventions:**
- Quote direction: $S^{\text{USD/EUR}}$ = USD per EUR (domestic per foreign)
- Spot: $S_0^{\text{USD/EUR}} = 1.2000$
- Continuous compounding: $r_{\text{USD}} = 5\%$, $r_{\text{EUR}} = 2\%$, $T = 0.5$ years
- Use Hull interest-parity form: $F_0 = S_0 e^{(r - r_f)T}$

**Step 1: Invert the spot quote**
$$S_0^{\text{EUR/USD}} = \frac{1}{1.2000} = 0.833333 \text{ EUR per USD}$$

**Step 2: Compute the forward in USD/EUR**
$$F_0^{\text{USD/EUR}} = 1.2000 \times e^{(0.05 - 0.02) \times 0.5} = 1.2000 \times e^{0.015}$$

Using $e \approx 2.718281828$:
$$e^{0.015} = 2.718281828^{0.015} \approx 1.0151130646$$

so:
$$F_0^{\text{USD/EUR}} \approx 1.2000 \times 1.0151130646 = 1.2181356775$$

**Step 3: Invert the forward**
$$F_0^{\text{EUR/USD}} = \frac{1}{1.2181356775} \approx 0.8209266163$$

**Step 4: Verify via the inverted-quote formula**

When you invert, the interest differential flips:
$$F_0^{\text{EUR/USD}} = S_0^{\text{EUR/USD}} \, e^{(r_{\text{EUR}} - r_{\text{USD}})T} = 0.833333 \times e^{-0.015} \approx 0.8209266$$

which matches. $\checkmark$

**Unit checks:**
- $S^{\text{USD/EUR}}$: USD/EUR
- $F^{\text{USD/EUR}}$: USD/EUR
- Inverse quotes are EUR/USD $\checkmark$

---

### Example B — CIP Forward from Simple Rates (Rate-Based and DF-Based; Verify Match)

**Conventions:**
- Quote direction: USD per EUR
- Spot: $S_0 = 1.2000$ USD/EUR
- Simple annualized rates over year fraction $T = 0.25$:
  - $R_D = 4\%$ (USD), $R_F = 1\%$ (EUR)

**(1) Rate-based simple formula:**
$$F_0 = S_0 \frac{1 + R_D T}{1 + R_F T} = 1.2000 \times \frac{1 + 0.04 \times 0.25}{1 + 0.01 \times 0.25}$$

Compute:
- $1 + 0.04 \times 0.25 = 1.01$
- $1 + 0.01 \times 0.25 = 1.0025$

So:
$$F_0 = 1.2000 \times \frac{1.01}{1.0025} = 1.2089775561 \text{ USD/EUR}$$

**(2) Discount-factor formula:**
$$P_D(0,T) = \frac{1}{1 + R_D T} = \frac{1}{1.01} = 0.9900990099$$
$$P_F(0,T) = \frac{1}{1 + R_F T} = \frac{1}{1.0025} = 0.9975062344$$

Then:
$$F_0 = S_0 \frac{P_F(0,T)}{P_D(0,T)} = 1.2000 \times \frac{0.9975062344}{0.9900990099} = 1.2089775561$$

**Verification:** Matches exactly under consistent conventions. $\checkmark$

---

### Example C — Forward Points $F - S$; Sign Interpretation

Using Example B:
$$\text{Forward points} = F_0 - S_0 = 1.2089775561 - 1.2000 = 0.0089775561 \text{ USD/EUR}$$

**Interpretation (in USD/EUR quote):**
Points are positive because domestic (USD) rate $4\% >$ foreign (EUR) rate $1\%$, so $P_D < P_F \Rightarrow F > S$.

**Caution:** Points sign flips if you invert the quote (EUR/USD). (See Example A logic.)

---

### Example D — PV of an FX Forward from $F_{\text{mkt}}$ vs Fair $F^*$

**Conventions:**
- Notional: $N_F = 10{,}000{,}000$ EUR
- Maturity: $T = 0.25$
- Spot: $S_0 = 1.2000$ USD/EUR
- Discount factors: from Example B, $P_D = 0.9900990099$, $P_F = 0.9975062344$
- Fair forward: $F^* = 1.2089775561$ USD/EUR
- Market traded forward (delivery price): $K = 1.2100$ USD/EUR

**PV in domestic currency:**
$$V_0^{(\text{USD})} = N_F \left( S_0 P_F - K P_D \right)$$

Compute:
- $S_0 P_F = 1.2000 \times 0.9975062344 = 1.1970074813$
- $K P_D = 1.2100 \times 0.9900990099 = 1.1980198020$

Difference:
$$S_0 P_F - K P_D = -0.0010123207 \text{ USD per EUR}$$

So:
$$V_0^{(\text{USD})} = 10{,}000{,}000 \times (-0.0010123207) = -10{,}123.21 \text{ USD}$$

**Sign for a long-forward position:**
- Long forward = buy EUR at $K$
- If $K > F^*$, you agreed to pay too many USD per EUR relative to no-arbitrage, so PV is negative $\checkmark$

---

### Example E — Replicating Strategy / No-Arbitrage (Explicit Borrow/Lend Cashflows)

Use Example B data but assume the market forward is mispriced high:
- $S_0 = 1.2000$ USD/EUR
- $T = 0.25$
- $R_{\text{USD}} = 4\%$ simple, $R_{\text{EUR}} = 1\%$ simple
- Market forward: $F_{\text{mkt}} = 1.2200$ USD/EUR (too high vs CIP 1.20898)

**Arbitrage strategy (when forward is too high):**

1. **Borrow USD 1.2000 now.**
   Repay at $T$:
   $$1.2000 (1 + 0.04 \times 0.25) = 1.2120 \text{ USD}$$

2. **Buy EUR at spot:** 1.2000 USD buys 1 EUR

3. **Invest EUR at 1% simple for 0.25y:**
   $$1 \times (1 + 0.01 \times 0.25) = 1.0025 \text{ EUR at } T$$

4. **Sell EUR forward at $F_{\text{mkt}} = 1.2200$:** deliver 1.0025 EUR at $T$, receive:
   $$1.0025 \times 1.2200 = 1.22305 \text{ USD}$$

**Net at maturity $T$:**
- Receive: 1.22305 USD
- Repay loan: 1.2120 USD
- **Profit: 0.01105 USD** per 1 EUR initial scale

Scaled to 1,000,000 EUR hedge size: profit $= 11{,}050$ USD.

**Conclusion:** No-arbitrage forces $F_{\text{mkt}}$ back to CIP-implied level.

---

### Example F — Strip Across Maturities: Build a Forward Curve from Two DF Curves

**Conventions:**
- Spot $S_0 = 1.2000$ USD/EUR
- Domestic discount factors $P_D(0,T)$:
  - $T = 0.25$: 0.9900
  - $T = 0.50$: 0.9800
  - $T = 1.00$: 0.9600
- Foreign discount factors $P_F(0,T)$:
  - $T = 0.25$: 0.9950
  - $T = 0.50$: 0.9900
  - $T = 1.00$: 0.9750

Use DF-form CIP:
$$F(0,T) = S_0 \frac{P_F(0,T)}{P_D(0,T)}$$

**Compute forwards:**

$T = 0.25$:
$$F(0, 0.25) = 1.2 \times \frac{0.995}{0.99} = 1.2060606061$$

$T = 0.50$:
$$F(0, 0.50) = 1.2 \times \frac{0.99}{0.98} = 1.2122448980$$

$T = 1.00$:
$$F(0, 1.00) = 1.2 \times \frac{0.975}{0.96} = 1.2187500000$$

**Interpretation:**
The forward curve is upward sloping in USD/EUR because domestic discounting is "faster" (lower $P_D$) than foreign.

---

### Example G — Hedging a Foreign Bond with an FX Forward (Domestic PV and Hedged Return)

**Conventions:**
- You buy a foreign zero-coupon bond paying $N_F = 1{,}000{,}000$ EUR at $T = 1$
- Foreign DF: $P_F(0,1) = 0.9750$
- Domestic DF: $P_D(0,1) = 0.9600$
- Spot: $S_0 = 1.2000$ USD/EUR
- Hedge: short FX forward to sell 1,000,000 EUR at $T = 1$ at fair forward $K = F(0,1) = 1.21875$ (from Example F)

**Step 1: Foreign PV (in EUR)**
$$PV^{(\text{EUR})} = 1{,}000{,}000 \times 0.9750 = 975{,}000 \text{ EUR}$$

**Step 2: Domestic PV of unhedged position**

Convert at spot:
$$PV_{\text{unhedged}}^{(\text{USD})} = S_0 \times PV^{(\text{EUR})} = 1.2 \times 975{,}000 = 1{,}170{,}000 \text{ USD}$$

**Step 3: Domestic PV of hedged position**

At $T$: bond pays 1,000,000 EUR, deliver into forward, receive:
$$\text{USD payoff at } T = K \times 1{,}000{,}000 = 1{,}218{,}750 \text{ USD}$$

Discount:
$$PV_{\text{hedged}}^{(\text{USD})} = 1{,}218{,}750 \times 0.9600 = 1{,}170{,}000 \text{ USD}$$

**Result:**
Hedged PV equals unhedged PV when $K$ is the fair forward (CIP consistency).

**Hedged return interpretation:**
$$\frac{1{,}218{,}750}{1{,}170{,}000} = 1.0416667 = \frac{1}{0.96}$$

So the hedged foreign ZCB behaves like a domestic ZCB (you earn the domestic discounting return), consistent with "covered" parity logic.

---

### Example H — Rolling Hedge Carry Over 3 Months (Toy Forward Points)

**Goal:** Illustrate carry from forward points when spot is flat.

**Conventions:**
- You are long a foreign exposure of 1,000,000 EUR over 3 months
- You hedge by selling EUR forward in 1-month pieces, rolling monthly
- Assume for simplicity:
  - Spot stays constant at $S = 1.2000$ at each monthly expiry
  - Each 1-month forward you sell is $K = 1.2020$ USD/EUR (i.e., +0.0020 forward points)
  - Ignore transaction costs and curve changes

**Month 1:**
- Short 1M forward at $K = 1.2020$
- At expiry, spot is 1.2000
- Short-forward payoff in USD (domestic-equivalent):
$$N_F(K - S) = 1{,}000{,}000 \times (1.2020 - 1.2000) = 2{,}000 \text{ USD}$$

**Month 2 and 3:**
Same assumptions → each month earns 2,000 USD.

**Total 3-month carry from rolling:**
$$2{,}000 + 2{,}000 + 2{,}000 = 6{,}000 \text{ USD}$$

**Interpretation:**
This illustrates that (under flat spot) the hedger earns the forward points embedded by the interest differential.

---

### Example I — Effect of Compounding Mismatch: "Fake Arbitrage" If You Mix Conventions

**Setup:**
- Spot $S_0 = 1.2000$ USD/EUR
- Horizon $T = 2$ years
- Quoted simple annual rates (for the 2Y horizon):
  - Domestic $R_D = 10\%$
  - Foreign $R_F = 2\%$

**Step 1: Correct simple-rate forward**
$$F_{\text{simple}} = S_0 \frac{1 + R_D T}{1 + R_F T} = 1.2 \times \frac{1 + 0.10 \times 2}{1 + 0.02 \times 2} = 1.2 \times \frac{1.20}{1.04} = 1.3846153846$$

**Step 2: Wrong continuous formula if you (incorrectly) treat the same numbers as continuous**

Use Hull form $F = S e^{(r - r_f)T}$ but plug $r = 10\%$, $r_f = 2\%$:
$$F_{\text{wrong}} = 1.2 \times e^{(0.10 - 0.02) \times 2} = 1.2 \times e^{0.16} \approx 1.2 \times 1.1735108710 = 1.4082130452$$

**Observation:**
The two "forwards" differ materially:
$$1.4082 - 1.3846 \approx 0.0236 \text{ USD/EUR}$$

This is **not a real arbitrage**; it is a convention mismatch (simple vs continuous compounding).

**Practitioner lesson:**
Use discount factors $P_D, P_F$ consistently (or convert rates properly) to avoid false signals. The DF-ratio formula is convention-robust:
$$F = S_0 \frac{P_F}{P_D}$$

---

### Example J — Triangular Arbitrage Check (Optional)

**I'm not sure.** The provided sources do not explicitly discuss triangular arbitrage checks for spot FX quotes. The following is a generic no-arbitrage consistency check.

**Given spot quotes (all as USD per currency):**
- $S(\text{USD/EUR}) = 1.2000$
- $S(\text{USD/GBP}) = 1.5000$

**Implied cross rate (GBP per EUR):**
$$S(\text{GBP/EUR})_{\text{implied}} = \frac{\text{USD/EUR}}{\text{USD/GBP}} = \frac{1.2000}{1.5000} = 0.8000 \text{ GBP per EUR}$$

If the market actually quotes $S(\text{GBP/EUR}) = 0.7900$, then the three quotes are inconsistent, potentially allowing a triangular loop (ignoring bid/ask and settlement).

---

## Practical Notes

### 5.1 Quoting and Operational Gotchas

**Spot date conventions (T+2, T+1, etc.):**
**I'm not sure.** The provided sources do not specify FX spot settlement rules by currency pair. To be precise, we need the pair and market convention (e.g., standard spot date, holidays).

General caution: "spot" in some markets does not mean same-day settlement; Tuckman notes most fixed-income "spot" trades settle in one or two days and "true" spot would be same-day.

**Bid/ask:**
Forward points and spot both have bid/ask; using mid for PV is an approximation. Forward pricing identities are no-arbitrage statements in frictionless settings; in practice, costs widen bands.

**Forward points scaling (pips/points):**
**I'm not sure.** Not covered in sources; scaling is pair-dependent and must be checked with desk standards.

**Deliverable vs NDF:**
**I'm not sure.** Not covered in sources; NDF settlement conventions (net cash settlement, usually in one currency) change operational cashflows and sometimes curve inputs.

---

### 5.2 Common Pitfalls

1. **Mixing quote direction:** Forgetting to invert the DF ratio when inverting the spot quote
2. **Mixing compounding conventions:** Example I shows how "fake arbitrage" arises from inconsistent compounding assumptions
3. **Mixing rates vs discount factors:** Use $P_D, P_F$ when valuing PV; don't plug "rates" without specifying compounding and year fractions
4. **Forgetting the "fair forward" depends on which curves are used** (preview: multi-curve and cross-currency basis effects)

---

### 5.3 Implementation Pitfalls

**Date generation / year-fraction:**
Ensure $T$ matches the money-market conventions used to build $P_D, P_F$.

**Curve consistency:**
Use a consistent set of domestic and foreign discount factors when valuing both legs; otherwise $V(K = F^*) \neq 0$ and you create spurious PV.

**Handling spot-lag explicitly:**
If the market defines spot as settling at $t_{\text{spot}} \neq t$, then the theoretical "spot" in formulas should be the spot-settlement FX consistent with the cashflow dates.

---

### 5.4 Verification Tests (Desk Checklist)

| Test | Check |
|------|-------|
| **Dimension/unit checks** | $S, F, K$ must be in the same units (e.g., USD/EUR). $P_D, P_F$ are dimensionless. |
| **CIP replication check** | Reproduce $F(t,T)$ from borrowing/lending and spot conversion; confirm no arbitrage. |
| **PV test** | Verify $V_t(K = F^*) = 0$ when curves and quotes are consistent. |
| **Sensitivity sanity** | In $D/F$ quotes, higher domestic rates (lower $P_D$) increase $F = S(P_F / P_D)$; higher foreign rates (lower $P_F$) decrease $F$. (All else equal.) |

---

## Summary & Recall

### 10-Bullet Executive Summary

1. **FX spot** $S_t^{D/F}$ is domestic currency per 1 foreign; inverse quote is $1/S$
2. **FX forwards** exchange two currencies at maturity: receive foreign, pay domestic at delivery price $K$
3. **No-arbitrage "covered interest parity"** links spot and forward through interest differentials (Hull's interest rate parity)
4. **Treat foreign currency** as an asset earning the foreign risk-free rate (like a yield)
5. **DF form:** $F(t,T) = S_t \frac{P_F(t,T)}{P_D(t,T)}$ (quote direction matters)
6. **Domestic PV of a forward:** $V_t = N_F(S_t P_F - K P_D)$; equivalently $V_t = N_F P_D(F^* - K)$
7. **Forward points** $F - S$ reflect interest differential and time; quoting conventions and scaling are pair-specific (not sourced here)
8. **Hedging known foreign cashflows:** Selling foreign forward locks domestic value (idealized)
9. **Rolling hedges** create P&L from spot moves, carry (points), and curve changes (stack-and-roll style)
10. **Across maturities,** CIP links the entire forward curve to two discount curves; beyond ~1Y, cross-currency basis swaps are key instruments (preview)

---

### Cheat Sheet of Formulas

| Identity | Formula |
|----------|---------|
| **Quote inversion** | $S^{F/D} = \frac{1}{S^{D/F}}, \quad F^{F/D}(t,T) = \frac{1}{F^{D/F}(t,T)}$ |
| **CIP / DF form** (domestic per foreign) | $F(t,T) = S_t \frac{P_F(t,T)}{P_D(t,T)}$ |
| **CIP / continuous-rate form** (Hull) | $F_0 = S_0 e^{(r - r_f)T}$ |
| **FX forward PV** (domestic currency) | $V_t^{(D)} = N_F(S_t P_F(t,T) - K P_D(t,T)) = N_F P_D(t,T)(F^*(t,T) - K)$ |
| **Forward points** (generic) | $\text{Pts} = F - S$ (**I'm not sure** about scaling conventions) |
| **Spot delta** (flat-yield intuition) | $\frac{\partial V}{\partial S} = N_F P_F(t,T)$ |

---

### Flashcards (30 Q/A)

| # | Question | Answer |
|---|----------|--------|
| 1 | What does $S_t^{D/F}$ mean? | Units of domestic currency per 1 unit of foreign currency |
| 2 | How do you invert an FX quote? | $S^{F/D} = 1/S^{D/F}$ |
| 3 | What are the cashflows of a long FX forward (buy foreign)? | Receive $N_F$ foreign; pay $K N_F$ domestic at maturity |
| 4 | What is "interest rate parity" for currency forwards in Hull? | $F_0 = S_0 e^{(r - r_f)T}$ |
| 5 | Why can foreign currency be treated like an asset with a yield? | Holding foreign currency earns the foreign risk-free rate, viewed as a yield |
| 6 | What is the DF-form CIP identity (domestic per foreign)? | $F = S(P_F / P_D)$ |
| 7 | What is $P_D(t,T)$? | Domestic ZCB price: PV in domestic currency of 1 domestic paid at $T$ |
| 8 | How do you convert a foreign ZCB PV into domestic PV? | Multiply by spot: $\tilde{P}_D = X(t) P_F(t,T)$ |
| 9 | What is the domestic PV of a long FX forward? | $N_F(S P_F - K P_D)$ |
| 10 | When is a forward worth zero at inception? | When $K$ equals the fair forward $F^*$ under consistent discounting |
| 11 | How do you express PV using $F^*$? | $V = N_F P_D(F^* - K)$ |
| 12 | If domestic rates rise (all else equal), what happens to $F$ in $D/F$ quotes? | $P_D$ decreases, so $F = S(P_F / P_D)$ increases |
| 13 | If foreign rates rise (all else equal), what happens to $F$ in $D/F$ quotes? | $P_F$ decreases, so $F$ decreases |
| 14 | What is the generic definition of forward points? | $F - S$ (not explicitly sourced) |
| 15 | What does "covered" mean in CIP? | The FX conversion at maturity is locked in via a forward |
| 16 | How does a firm hedge a known foreign inflow? | Sell the foreign currency forward to lock domestic value |
| 17 | What is the spot delta of an FX forward (in DF form)? | $N_F P_F(t,T)$ |
| 18 | What does Hull's $\Delta = e^{-qT}$ imply for FX forwards? | Set $q = r_f$ (foreign rate as yield) |
| 19 | What is "carry" in fixed income terms? | Expected return from holding a position assuming yields don't change |
| 20 | What drives FX forward "carry" in a simple view? | The interest differential embedded in $F/S$ |
| 21 | How does quote inversion affect CIP formulas? | It inverts both the spot and the DF ratio: $F^{F/D} = S^{F/D}(P_D / P_F)$ |
| 22 | What happens if you mix compounding conventions across currencies? | You can create apparent (fake) arbitrage signals (Example I) |
| 23 | How do you build a forward curve across maturities? | Compute $F(0, T_i) = S_0 (P_F(0, T_i) / P_D(0, T_i))$ for each $T_i$ |
| 24 | Why does cross-currency curve building care about FX forwards? | FX forwards tie domestic and foreign discounting via arbitrage constraints |
| 25 | What limitation of FX forwards does *Interest Rate Modeling* note? | FX forward market rarely liquid beyond ~1 year |
| 26 | What instrument is used beyond 1 year for cross-currency curve info? | Floating-floating cross-currency basis swaps (preview) |
| 27 | Relationship between a one-period basis swap and FX forward (per source)? | A one-period CRX basis swap is identical to an FX forward contract |
| 28 | What is a key verification test for an FX forward pricer? | PV equals zero when $K = F^*$ and curves are consistent |
| 29 | What is the main operational ambiguity not fixed in these notes? | Spot/forward settlement conventions and day-count conventions (not sourced) |
| 30 | Desk rule for this chapter? | Always specify quote direction and curve/discounting conventions before computing forwards/PV |

---

## Mini Problem Set

### Problems

1. Given $S^{D/F} = 1.2500$, compute $S^{F/D}$.

2. Show that if $F = S(P_F / P_D)$, then the inverse-quote forward satisfies $F^{F/D} = S^{F/D}(P_D / P_F)$.

3. With $S_0 = 1.10$, $P_D(0, 0.5) = 0.98$, $P_F(0, 0.5) = 0.995$, compute $F(0, 0.5)$.

4. For a long forward with $N_F = 5$ million, compute PV given $S_0 = 1.10$, $K = 1.12$, $P_D = 0.99$, $P_F = 0.995$.

5. Explain (in words) how to arbitrage if market forward $F_{\text{mkt}}$ is above CIP-implied forward.

6. Using simple rates $R_D, R_F$, derive $F = S(1 + R_D T) / (1 + R_F T)$.

7. Suppose domestic rates rise while foreign rates unchanged. Predict the sign of the change in $F$ in domestic-per-foreign quotes.

8. A domestic investor will receive 10 million foreign in 6 months. What forward position (long/short, notional) hedges FX risk?

9. Re-do problem 4 but express PV as $N_F P_D(F^* - K)$; compute $F^*$ first.

10. Given a term structure $P_D(0, T_i)$, $P_F(0, T_i)$, compute the forward curve and identify which maturities have the largest forward points.

11. Show that the hedged foreign ZCB payoff becomes deterministic in domestic currency when hedged with a forward.

12. Describe how bid/ask spreads in spot and forwards affect the no-arbitrage band around CIP.

13. A hedge is rolled monthly for a year. List the key P&L drivers you would monitor.

14. Explain why using inconsistent day-count conventions across currencies can create apparent arbitrage.

15. (Preview) What does it mean if FX forwards imply a persistent deviation from $F = S(P_F / P_D)$?

16. Design a simple unit test suite for an FX forward pricer (inputs, outputs, checks).

---

### Solution Sketches (First Few)

**Problem 1:**
$$S^{F/D} = \frac{1}{S^{D/F}} = \frac{1}{1.2500} = 0.8000$$

**Problem 2:**
Start with $F^{D/F} = S^{D/F} \frac{P_F}{P_D}$. Taking inverse:
$$F^{F/D} = \frac{1}{F^{D/F}} = \frac{1}{S^{D/F}} \cdot \frac{P_D}{P_F} = S^{F/D} \frac{P_D}{P_F}$$

**Problem 3:**
$$F(0, 0.5) = 1.10 \times \frac{0.995}{0.98} = 1.10 \times 1.0153061 = 1.1168367$$

**Problem 4:**
$$V = N_F(S_0 P_F - K P_D) = 5{,}000{,}000 \times (1.10 \times 0.995 - 1.12 \times 0.99)$$
$$= 5{,}000{,}000 \times (1.0945 - 1.1088) = 5{,}000{,}000 \times (-0.0143) = -71{,}500$$

**Problem 5:**
Borrow domestic, buy foreign at spot, invest foreign at foreign rate, sell foreign forward at inflated rate. Lock in profit equal to $(F_{\text{mkt}} - F^*)$ scaled by notional and discounting.

---

## Source Map

### (A) Verified Facts
| Fact | Source |
|------|--------|
| Interest rate parity: $F_0 = S_0 e^{(r-r_f)T}$ | Hull Ch 5 |
| Foreign currency as asset with yield $r_f$ | Hull Ch 5 |
| Forward delta $= e^{-qT}$ with $q = r_f$ for FX | Hull Ch 5 |
| Forward from numeraire ratio | Hull Ch 5 |
| Domestic value of foreign ZCB: $X(t) P_F(t,T)$ | Andersen & Piterbarg Vol 1 |
| FX forwards link discount curves via arbitrage | Andersen & Piterbarg Vol 1 Ch 6 |
| FX forward market liquidity <1Y | Andersen & Piterbarg Vol 1 |
| One-period xccy basis swap = FX forward | Andersen & Piterbarg Vol 1 |

### (B) Reasoned Inference
| Inference | Derivation |
|-----------|------------|
| DF-form CIP | Mapped from Hull's numeraire formula |
| PV formula for FX forward | Discounting each currency leg separately |
| Quote inversion inverts DF ratio | Algebraic inversion |
| Forward points formula | Subtract spot from CIP forward |

### (C) Speculation
| Item | Note |
|------|------|
| Forward points scaling (pips) | Not in sources |
| FX spot settlement conventions | Not in sources |
| NDF mechanics | Not in sources |
| Triangular arbitrage details | Not explicitly sourced |

---

*Cross-references: Ch 2 (discount factors), Ch 3 (zero/forward rates), Ch 21 (cross-currency curves), Ch 30 (cross-currency swaps)*
