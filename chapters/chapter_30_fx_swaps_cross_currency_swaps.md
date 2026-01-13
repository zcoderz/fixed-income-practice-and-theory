# Chapter 30: FX Swaps and Cross-Currency Swaps — Structure and Valuation Dependencies

---

## Fact Classification

### (A) Verified Facts (Source-Backed)

- FX forward parity in continuous compounding: $F_0 = S_0 e^{(r - r_f)T}$ (Hull)
- FX forward parity in discount-factor form: $X_{T_p}(T) = \frac{P_f(T, T_p)}{P_d(T, T_p)} X(T)$ (Andersen & Piterbarg)
- Currency swap structure: two principals (one in each currency) are exchanged at the beginning and end (Hull)
- Floating-for-floating currency swap: each side pays a floating index on its principal, possibly with a spread added (Hull)
- Cross-currency basis swaps exchange floating Libor payments "plus or minus a spread"; notionals exchanged at inception and maturity with ratio set to spot FX (Andersen & Piterbarg)
- Forward floating rate from projection curve: $L_c(0, t_i, t_{i+1}) = \frac{1}{\tau_i}\left(\frac{P_c^{(L)}(0, t_i)}{P_c^{(L)}(0, t_{i+1})} - 1\right)$ (Andersen & Piterbarg)
- One-period cross-currency basis swap is identical to an FX forward contract (Andersen & Piterbarg)
- FX forwards are "rarely liquid beyond maturities of one year" (Andersen & Piterbarg)
- In collateralized inter-dealer markets, discounting may be tied to collateral rate (e.g., Fed funds); Libor is not a good proxy for risk-free discounting in collateralized trades (Andersen & Piterbarg)
- DV01 expression: $\text{DV01} = -D \times P \times 0.0001$ (Tuckman)

### (B) Reasoned Inference (Derived from A)

- CIP in discount-factor form at time 0: $F(0,T) = X(0) \frac{P_f(0,T)}{P_d(0,T)}$ (derived from Andersen & Piterbarg formula)
- PV of FX forward (receive 1 foreign, pay $K$ domestic at $T$): $\text{PV}_0^{(d)} = X(0) P_f(0,T) - K P_d(0,T)$ (derived via replication argument)
- Foreign cashflow conversion equivalence under CIP: $P_d(0,T) F(0,T) C_f(T) = X(0) P_f(0,T) C_f(T)$ (algebra from CIP-DF)
- FX delta for $\text{PV} = \text{PV}_d - X(0) \text{PV}_f^{(f)}$: $\frac{\partial \text{PV}}{\partial X(0)} = -\text{PV}_f^{(f)}$ (calculus)
- Basis DV01 structure: $\frac{\partial \text{PV}}{\partial b} \approx -X(0) N_f \sum_i \tau_i P_f(0, t_{i+1})$ (derived from additive basis in PV formula)

### (C) Speculation (Clearly Labeled; Minimal)

- I'm not sure whether the books provide a single canonical "FX swap" legal definition; the interest-rate modeling reference emphasizes that a one-period cross-currency basis swap is identical to an FX forward and that cross-currency basis swaps include exchanges of notionals at inception and maturity.
- I'm not sure a universal sign convention for basis is specified in the provided excerpts. In practice, desks differ on whether the quote is "add $+b$" to the foreign leg, domestic leg, or "basis on the non-USD leg vs USD."
- I'm not sure which XCCY variants are assumed in the sources beyond "exchange at beginning and end" and the basis-swap description; common market variants include "resettable" notionals and no-initial-exchange structures.
- I'm not sure the provided excerpts give a full multi-currency "collateral currency" discounting formula (e.g., how $F(0,T)$ changes when collateral is posted in a third currency).
- I'm not sure the sources define a single canonical "FX swap implied funding spread" metric with exact compounding and collateral conventions.
- I'm not sure the provided excerpts specify calendar/rolling conventions for these products.

---

## Conventions & Notation

### Notation Glossary

| Symbol | Meaning |
|--------|---------|
| $d$ | Domestic currency (valuation/reporting currency) |
| $f$ | Foreign currency |
| $X(t)$ or $X(0)$ | Spot FX rate at time $t$, quoted as domestic per 1 unit of foreign (e.g., USD per EUR) |
| $F(0,T)$ or $X_T(0)$ | Forward FX rate for exchange at time $T$, same quote direction as $X$ |
| $P_d(0,T)$ | Domestic discount factor to $T$ (dimensionless) |
| $P_f(0,T)$ | Foreign discount factor to $T$ (dimensionless) |
| $P_c^{(L)}(0,T)$ | Pseudo-discount (projection) curve for floating index in currency $c$ |
| $L_c(0, t_i, t_{i+1})$ | Forward floating rate for currency $c$ over $[t_i, t_{i+1}]$, implied from $P_c^{(L)}$ |
| $\tau_i$ | Accrual year fraction for period $[t_i, t_{i+1}]$ |
| $N_d$, $N_f$ | Domestic and foreign notionals |
| $b$ (or $e$) | Cross-currency basis spread applied to one floating leg |
| $\text{PV}$ | Present value in domestic currency unless stated otherwise |
| $\text{PV01}$ | PV change for a +1bp shift in a specified curve |

### Defaults Used in Examples

- **FX quote direction:** $X =$ domestic per 1 foreign (e.g., USD per EUR)
- **Valuation approach:** Discount factors used to avoid day-count/compounding ambiguity
- **Rate conversion (toy):** Simple annual compounding $P(0,T) = \frac{1}{1 + rT}$ (explicitly labeled, not a market convention claim)
- **Credit/XVA:** Out of scope, consistent with the interest-rate modeling text's warning about complexity

**Warning:** Some currencies are commonly quoted the other way around in practice; quote direction must be checked.

---

## Core Concepts

### 1) FX Forward (Building Block)

**Formal Definition:**
An FX forward contract sets a future exchange of currencies at a pre-agreed rate $F(0,T)$. In no-arbitrage models with domestic rate $r$ and foreign rate $r_f$, a standard parity result is:

$$F_0 = S_0 e^{(r - r_f)T}$$

where $S_0$ is the spot exchange rate (quoted as domestic per foreign).

**Intuition:**
If domestic interest rates exceed foreign interest rates, the foreign currency tends to trade at a forward premium in domestic-per-foreign terms (forward > spot), because holding foreign currency (low yield) must be compensated via the FX forward.

**Trading / Risk / Portfolio Practice:**
FX forwards are used to hedge future foreign-currency cashflows (e.g., a foreign coupon) or to lock funding across currencies.

---

### 2) Covered Interest Parity (CIP) / FX Forward Parity

**Formal Definition:**
In the multi-currency modeling reference, the forward FX rate is linked to discount factors as:

$$\boxed{X_{T_p}(T) = \frac{P_f(T, T_p)}{P_d(T, T_p)} X(T)}$$

i.e., forward equals spot multiplied by the ratio of foreign to domestic discount factors (same quote direction).

**Intuition:**
A currency-forward is pinned down by a cash-and-carry argument: borrow in one currency, invest in the other, and hedge FX with a forward; no arbitrage forces consistency.

**Trading / Risk / Portfolio Practice:**
Traders check whether the observed forward curve is consistent with curve inputs; persistent deviations motivate introducing a cross-currency basis framework (see below).

---

### 3) FX Swap

**Formal Definition:**
For this chapter's valuation purposes, an FX swap is treated as a spot FX exchange combined with an opposite-direction FX forward (equivalently, two exchanges of notionals at different dates: near date and far date).

I'm not sure whether the books provide a single canonical "FX swap" legal definition; the interest-rate modeling reference instead emphasizes that a one-period cross-currency basis swap is identical to an FX forward contract and that cross-currency basis swaps include exchanges of notionals at inception and maturity.

*What would be needed to be certain:* the exact market/documentation definition (e.g., which product standard, settlement conventions for the currency pair).

**Intuition:**
An FX swap is a funding instrument: it lets you borrow one currency and lend the other while locking the future FX conversion.

**Trading / Risk / Portfolio Practice:**
Used for short-dated funding and hedging. Interbank FX forward markets may be liquid mainly at shorter maturities (the reference notes FX forwards are "rarely liquid beyond maturities of one year").

---

### 4) Currency Swap / Cross-Currency Interest Rate Swap (XCCY)

**Formal Definition:**
A currency swap specifies two principals (one in each currency) and they are exchanged at the beginning and at the end of the swap's life.

In a floating-for-floating currency swap, each leg pays a floating rate on its currency principal, possibly with a spread added.

**Intuition:**
A long-dated currency swap transforms both:
- Currency of principal/cashflows, and
- Interest-rate index exposure (e.g., USD floating vs EUR floating)

**Trading / Risk / Portfolio Practice:**
Used for long-dated funding, asset/liability transformation, and hedging foreign-rate exposure. Hull notes valuation by: (i) assume future floating rates equal today's forward rates, (ii) discount each currency's cashflows at that currency's zero curve, (iii) convert PVs at the current exchange rate.

---

### 5) Cross-Currency Basis Swap and Basis Spread

**Formal Definition:**
The interest-rate modeling reference defines floating-floating cross-currency basis swaps as contracts exchanging floating Libor payments in one currency for floating Libor payments in another currency **plus or minus a spread**; they involve exchanging notionals at inception and maturity with the notional ratio set to the prevailing spot FX.

The spread (called $e$ in the USD/JPY example) is market-quoted and can appear on one leg, e.g., "USD Libor $+ x$" vs "JPY Libor $+ e + x$."

**Intuition:**
The basis spread compensates for funding/credit/liquidity imbalances across currency money markets; the text discusses episodes where the "CRX yield spread" became large in magnitude (e.g., around -40bp in late-1990s JPY; +60bp in early 2008).

**Trading / Risk / Portfolio Practice:**
Basis swaps are used to extend cross-currency information to long maturities; the reference notes FX forwards alone are too short-dated for building long yield curves, so cross-currency basis swaps (quoted out to long horizons) are used instead.

---

## Math and Derivations

### 2.1 CIP in Discount-Factor Form (and Unit Checks)

Assume:
- $X(0)$: spot FX, domestic/foreign
- $P_d(0,T)$, $P_f(0,T)$: discount factors

From the multi-currency formula in the reference:

$$\boxed{F(0,T) = X_T(0) = \frac{P_f(0,T)}{P_d(0,T)} X(0)} \tag{CIP-DF}$$

This is the discount-factor version of Hull's continuous-compounding parity $F_0 = S_0 e^{(r - r_f)T}$.

**Unit check:**
- $P_f / P_d$ is dimensionless
- $X(0)$ is domestic/foreign
- So $F(0,T)$ is domestic/foreign ✓

---

### 2.2 PV of an FX Forward in Domestic Currency

Consider a forward that at $T$:
- Receives 1 unit of foreign currency $f$
- Pays $K$ units of domestic currency $d$

Time-0 PV in domestic currency:

$$\boxed{\text{PV}_0^{(d)} = X(0) P_f(0,T) - K P_d(0,T)} \tag{FXF-PV}$$

**Reasoning:**
- "Receive 1 foreign at $T$" is equivalent to holding a foreign ZCB paying 1 $f$ at $T$, priced $P_f(0,T)$ in $f$; converting at spot gives domestic price $X(0) P_f(0,T)$
- "Pay $K$ domestic at $T$" is a domestic ZCB liability of PV $K P_d(0,T)$

**Fair forward rate (PV = 0):**

$$0 = X(0) P_f(0,T) - K^\star P_d(0,T) \implies K^\star = X(0) \frac{P_f(0,T)}{P_d(0,T)} = F(0,T)$$

which matches (CIP-DF).

---

### 2.3 FX Swap as Spot + Forward (Funding Interpretation)

Model an FX swap (in "domestic per foreign" quote) as:
- **Near-date:** exchange $N_f$ foreign for $N_d = X(0) N_f$ domestic at spot
- **Far-date $T$:** exchange back at forward $F(0,T)$

**Cashflows (domestic perspective):**
- At near-date (treat as time 0 for toy valuation): $-N_d + X(0) N_f = 0$
- At $T$: receive $F(0,T) N_f$ domestic and pay $N_f$ foreign

Thus, the economic value is primarily the forward leg value; under no-arbitrage $F(0,T) = K^\star$ makes the full package "fair" at inception.

**Funding interpretation (direction matters):**
- If you pay domestic now and receive domestic later, you are effectively **lending domestic**
- If you receive foreign now and repay foreign later, you are effectively **borrowing foreign**

So an FX swap can synthetically implement "borrow one, lend the other," locked by FX.

---

### 2.4 Cross-Currency Basis Swap PV (Source-Backed Formula) and Dependencies

The interest-rate modeling reference provides an explicit PV for a USD/JPY floating-floating basis swap where a USD-based party receives USD Libor flat and pays JPY Libor plus spread $e_¥$, including principal exchange at maturity (with notional ratio tied to spot). For notional \$1, the USD PV is given as:

$$\boxed{
\begin{aligned}
V_{\text{basisswap},\$}(0) &= \sum_{i=0}^{n-1} L_\$(0, t_i, t_{i+1}) \tau_i P_\$(0, t_{i+1}) + P_\$(0, t_n) \\
&\quad - X(0) \left( \sum_{i=0}^{n-1} (L_¥(0, t_i, t_{i+1}) + e_¥) \tau_i P_¥(0, t_{i+1}) + P_¥(0, t_n) \right)
\end{aligned}
} \tag{XCCY-PV}$$

They then rewrite it using the projection curve $P_¥^{(L)}$ and the identity between USD discount and USD projection curves (Assumption 6.5.1) to simplify the USD floating leg PV to \$1.

**What this formula makes unambiguous ("what curve does what?"):**

| Curve | Role |
|-------|------|
| **Discounting** $P_\$$ | Discounts USD cashflows |
| **Discounting** $P_¥$ | Discounts JPY cashflows |
| **Projection** $P_¥^{(L)}$ | Generates forward JPY Libor via $L_¥(0, t_i, t_{i+1}) = \frac{1}{\tau_i}\left(\frac{P_¥^{(L)}(0, t_i)}{P_¥^{(L)}(0, t_{i+1})} - 1\right)$ |
| **Spot FX** $X(0)$ | Converts foreign PV into domestic PV |
| **Par basis spread** $e_¥^{par}$ | Market quote that sets $V_{\text{basisswap},\$}(0) = 0$ |

---

### 2.5 Generalized Dependency Statement (Derived)

Even when you do not adopt Assumption 6.5.1 (i.e., even if discount and projection curves differ in both currencies), the valuation dependency structure remains:

$$\text{PV}^{(d)} = \underbrace{\text{PV of domestic leg discounted on domestic curve}}_{\text{domestic projection + discount}} - X(0) \times \underbrace{\text{PV of foreign leg discounted on foreign curve}}_{\text{foreign projection + discount}}$$

where:
- Domestic floating cashflows require **domestic projection curve** to forecast $L_d$ and **domestic discount curve** $P_d$ to PV
- Foreign floating cashflows require **foreign projection curve** and **foreign discount curve**
- Conversion into domestic currency uses **spot FX** (or equivalently forward FX plus domestic discounting, if the FX forwards are consistent with the curves)

**Sanity check:** If FX forwards satisfy CIP in the curve set, converting each foreign cashflow $C_f(T)$ via forward $F(0,T)$ and discounting domestically is equivalent to spot conversion and foreign discounting:

$$P_d(0,T) F(0,T) C_f(T) = P_d(0,T) \left( X(0) \frac{P_f(0,T)}{P_d(0,T)} \right) C_f(T) = X(0) P_f(0,T) C_f(T)$$

This is just algebra from (CIP-DF).

---

### 2.6 Collateral/Discounting Preview (Source-Backed Boundary)

The reference emphasizes that in collateralized inter-dealer markets, discounting may be tied to the collateral rate (e.g., Fed funds), and that Libor is not a good proxy for risk-free discounting in such collateralized trades; this motivates OIS discounting.

I'm not sure the provided excerpts give a full multi-currency "collateral currency" discounting formula (e.g., how $F(0,T)$ changes when collateral is posted in a third currency).

*What would be needed:* the CSA collateral currency, collateral remuneration rate, and the exact framework/section treating multi-currency collateralization.

---

## Measurement & Risk (Chapter 30 Scope)

This chapter focuses on **structure + valuation dependencies** and makes "what curve does what?" unambiguous.

### A) FX Swap (Funding Instrument)

**Define:** An FX swap as spot + forward (or two exchanges of notionals at different dates).

**Practical representation:** Spot exchange now and reverse exchange at $T$ at a pre-agreed forward rate.

**Source anchor:** The reference explicitly notes a one-period cross-currency basis swap is identical to an FX forward, and basis swaps exchange notionals at inception and maturity.

**Explain the funding interpretation:** Borrowing one currency and lending the other, locked by FX.

**Directional interpretation:**
- Spot-buy foreign + forward-sell foreign $\Rightarrow$ borrow foreign, lend domestic (domestic outflow now, domestic inflow later)
- Spot-sell foreign + forward-buy foreign $\Rightarrow$ borrow domestic, lend foreign

**Show the no-arbitrage link to CIP:**

*Discount-factor parity:*
$$F(0,T) = X(0) \frac{P_f(0,T)}{P_d(0,T)}$$
(multi-currency reference)

*Rate-parity form (continuous compounding):*
$$F_0 = S_0 e^{(r - r_f)T}$$
(derivatives reference)

**Clarify what curve objects enter valuation:**
- Domestic discount curve $P_d(0,T)$
- Foreign discount curve $P_f(0,T)$
- Spot FX $X(0)$
- Forward FX $F(0,T)$ (either observed directly or implied from the curves via CIP)

---

### B) Cross-Currency Swap (XCCY / Basis Swap)

**Define:** A cross-currency swap as exchanging floating-rate legs in two currencies (often with notional exchanges; state alternatives and label).

**Currency swap structure:** Two principals (one each currency) are exchanged at the beginning and end.

**Floating-floating variant:** Each side pays a floating index on its principal, possibly with a spread added.

**Alternative structures (market variants):**
I'm not sure which variants are assumed in the provided sources beyond "exchange at beginning and end" and the basis-swap description; common market variants include "resettable" notionals and no-initial-exchange structures, but these are not confirmed in the provided excerpts.

*What would be needed:* product standard and the specific section defining notional exchange rules for your currency pair.

**Define the cross-currency basis spread:** What it is, where it appears (which leg gets spread), and sign conventions (flag differences).

**Source:** Basis swaps exchange floating Libor payments "plus or minus a spread."

The reference's example uses $e$ as the market-quoted basis added on one leg (JPY leg in the illustrative arbitrage scheme: "JPY Libor $+ e + x$").

**Sign conventions:**
I'm not sure a universal sign convention is specified in the provided excerpts. In practice, desks differ on whether the quote is:
- "add $+b$" to the foreign leg, or
- "add $+b$" to the domestic leg, or
- quote as "basis on the (non-USD) leg vs USD," etc.

*What would be needed:* the market quoting convention for the specific currency pair and which leg is defined as "pay/receive basis" in your confirmations.

**Explain curve dependencies in a modern framework:**

*Discounting curves (possibly collateral-currency dependent):*
- Currency swaps are valued by discounting each currency's cashflows on its own zero curve and converting with spot FX (Hull's stated procedure)
- In modern collateralized markets, discounting may be tied to collateral/OIS rather than Libor proxies

*Projection curves for each floating index (and tenor):*
- The reference explicitly separates discount curves $P$ and index projection curves $P^{(L)}$, and uses $P^{(L)}$ to generate forward Libor rates via discount-factor ratios

*FX forward curve consistency / arbitrage constraints:*
- The FX forward curve and cross-currency basis markets must be fit consistently to avoid arbitrage; the reference provides a concrete "zero-cost scheme" showing inconsistent curves imply arbitrage opportunities

---

### C) Dependency Graph and "What You're Long/Short"

**Provide a clear decomposition of exposures:**

#### FX Delta (Spot)

For a position with PV in domestic currency:
$$\text{PV}^{(d)} = \text{PV}_d - X(0) \text{PV}_f^{(f)}$$

Holding curves and cashflow definitions fixed:
$$\frac{\partial \text{PV}^{(d)}}{\partial X(0)} = -\text{PV}_f^{(f)}$$

**Intuition:** You are "short foreign PV" in domestic terms if you pay the foreign leg.

#### Domestic IR PV01 and Foreign IR PV01

Define PV01 as PV change under a +1bp shift to a specified curve (parallel bump). We use the DV01 idea (PV change per bp) as an analogue, but the "curve bump" definition must be made explicit in implementation.

#### Basis Sensitivity (Basis DV01)

If basis spread $b$ is an additive coupon spread on one leg, PV is approximately linear in $b$:

$$\frac{\partial \text{PV}}{\partial b} \approx -X(0) N_f \sum_i \tau_i P_f(0, t_{i+1})$$

for the common structure where paying foreign "Libor + $b$" adds $b \tau_i N_f$ at each coupon date.

(This linear structure is explicit in the source PV formula (XCCY-PV) where $e$ enters additively inside the discounted coupon sum.)

#### (Preview Only) Collateral Currency Effect on Discounting

Discount curve choice changes PV materially; the reference links collateralization to OIS discounting and warns against Libor-as-risk-free in collateralized trades.

I'm not sure the provided excerpts give a complete "hold collateral currency fixed vs change collateral currency" sensitivity definition; you need CSA details.

**State explicitly what is held fixed in each sensitivity/bump:**

In this chapter's toy risk decomposition examples we will:
- Hold projection curves fixed when bumping discount curves (to isolate discounting PV01)
- Hold spot FX fixed when bumping curves
- Hold basis spread fixed when bumping curves (unless we are computing basis DV01)

**Note:** In fully arbitrage-consistent multi-currency setups, bumping one curve may require re-implying forwards/basis; that is beyond scope here and belongs in a curve construction chapter.

---

### D) Keep It Focused

- Do NOT turn this into a full cross-currency curve-construction chapter (that's earlier), but you may reference the logic
  - We reference the no-arbitrage consistency logic and the source's arbitrage scheme, but we do not bootstrapping-build full curves here
- Do NOT turn it into a full XVA chapter; only mention collateral/CSA discounting insofar as it changes which curve discounts cashflows
  - We only preview that collateralized discounting is often OIS-linked; full CVA/XVA and CSA mechanics are outside scope (as the reference states)

---

## Worked Examples

### Global Toy Conventions for Examples A-L

| Parameter | Value |
|-----------|-------|
| Quote direction | $X =$ USD per EUR (domestic USD, foreign EUR) |
| Times | $T = 1$ year; for 2-period swaps: $t_1 = 0.5$, $t_2 = 1.0$ |
| Accruals | $\tau_1 = \tau_2 = 0.5$ |
| Rate conversion | Toy simple annual compounding $P(0,T) = 1/(1 + rT)$ (explicitly labeled) |
| Discount curves | $P_d$, $P_f$ discount cashflows in their currency |
| Projection curves | $P_d^{(L)}$, $P_f^{(L)}$ generate forward floating rates by the ratio formula |
| Notional exchange | Included at maturity; initial exchange at spot has zero PV in the toy setup |

---

### Example A: FX Swap as Spot + Forward — Cashflows and Funding Interpretation

**Inputs:**
- Spot: $X(0) = 1.10$ USD/EUR
- Forward: $F(0,1) = 1.1150$ USD/EUR
- Notional: $N_f = 10{,}000{,}000$ EUR

**Structure (spot + forward):**

*Near-date (treat as time 0 for toy):*
- Pay $N_d = X(0) N_f = 1.10 \times 10{,}000{,}000 = 11{,}000{,}000$ USD
- Receive $10{,}000{,}000$ EUR

*Far-date $T = 1$:* reverse exchange
- Pay $10{,}000{,}000$ EUR
- Receive $F(0,1) N_f = 1.1150 \times 10{,}000{,}000 = 11{,}150{,}000$ USD

**Funding interpretation (directional):**
- You pay USD now and receive USD later $\Rightarrow$ you are **lending USD**
- You receive EUR now and pay EUR later $\Rightarrow$ you are **borrowing EUR**
- The forward locks the EUR payback cost in USD

---

### Example B: CIP Fair Forward Using Discount Factors — Compute $F^\star(0,T)$, Compare to Market, Compute PV

**Inputs:**
- $X(0) = 1.10$ USD/EUR
- Domestic DF: $P_d(0,1) = 0.98$
- Foreign DF: $P_f(0,1) = 0.99$

**CIP fair forward (discount-factor form):**

$$F^\star(0,1) = X(0) \frac{P_f(0,1)}{P_d(0,1)} = 1.10 \times \frac{0.99}{0.98} = 1.11122449 \text{ USD/EUR}$$

**Unit check:**
- $P_f / P_d$ dimensionless; $F^\star$ has units USD/EUR ✓

**Compare to a "market forward":**

Suppose market forward quote $F_{\text{mkt}}(0,1) = 1.1150$ USD/EUR.

**PV of an FX forward mismatch:**

Consider a forward where you receive 10,000,000 EUR and pay $K = 1.1150$ USD/EUR at $T = 1$.

PV in USD:
$$\text{PV}_0 = X(0) P_f(0,1) N_f - K P_d(0,1) N_f$$

**Intermediate steps (per EUR):**
- $X(0) P_f = 1.10 \times 0.99 = 1.0890$
- $K P_d = 1.1150 \times 0.98 = 1.0927$
- Difference $= 1.0890 - 1.0927 = -0.0037$ USD/EUR

**Multiply by $N_f = 10{,}000{,}000$:**
$$\text{PV}_0 = -0.0037 \times 10{,}000{,}000 = -37{,}000 \text{ USD}$$

**Interpretation:** Paying a forward rate above fair ($K > F^\star$) makes the "buy EUR forward" position negative PV.

---

### Example C: Implied Foreign Discount Factor from FX Forward — Infer $P_f(0,T)$ and Implied Zero Rate

**Inputs:**
- $X(0) = 1.10$
- Market forward $F_{\text{mkt}}(0,1) = 1.1150$
- Domestic DF $P_d(0,1) = 0.98$

From CIP-DF:

$$P_f(0,1) = \frac{F_{\text{mkt}}(0,1)}{X(0)} P_d(0,1) = \frac{1.1150}{1.10} \times 0.98 = 0.99336364$$

**Toy implied foreign zero rate (simple annual compounding):**

Using $P_f(0,1) = \frac{1}{1 + r_f} \Rightarrow r_f = \frac{1}{P_f} - 1$:

$$r_f^{\text{(simple)}} = \frac{1}{0.99336364} - 1 = 0.00668070 \approx 0.6681\%$$

**Caution:** This rate conversion is illustrative; actual compounding/day count is market-specific.

---

### Example D: FX Swap Funding Rate Interpretation — Implied "All-In" Funding Spread (Labeled Intuition)

Using Example B/C inputs.

**Goal (funding intuition):** Compare "implied domestic rate" backed out from FX swap/forward vs domestic curve.

I'm not sure the sources define a single canonical "FX swap implied funding spread" metric with exact compounding and collateral conventions.

*What would be needed:* (i) exact money-market conventions for both currencies, (ii) whether discounting is OIS/CSA-based, (iii) whether you want unsecured vs collateralized funding interpretation.

**Toy calculation (simple compounding):**

Under simple-compounding parity:
$$\frac{F}{X} = \frac{1 + r_d}{1 + r_f} \Rightarrow r_d^{\text{implied}} = \frac{F}{X}(1 + r_f) - 1$$

**Domestic simple rate from domestic DF:**
$$P_d(0,1) = 0.98 \Rightarrow r_d^{\text{curve}} = \frac{1}{0.98} - 1 = 2.0408\%$$

**Foreign simple rate from foreign DF (baseline curve):**
$$P_f(0,1) = 0.99 \Rightarrow r_f = 1/0.99 - 1 = 1.0101\%$$

**Use market forward $F = 1.1150$, spot $X = 1.10$:**
$$r_d^{\text{implied}} = \left(\frac{1.1150}{1.10}\right)(1 + 0.010101) - 1 = 2.3875\%$$

**Implied spread vs domestic curve:**
$$\Delta r_d = r_d^{\text{implied}} - r_d^{\text{curve}} = 2.3875\% - 2.0408\% = 0.3467\% \approx 34.7 \text{ bp}$$

**Interpretation (toy):** The forward appears to embed an additional ~35bp differential relative to the baseline curves.

---

### Example E: PV of an FX Swap Off-Market — Discount Both Currency Cashflows

Use same $X(0) = 1.10$, $P_d(0,1) = 0.98$, and fair forward $F^\star = 1.11122449$.

**Trade definition (FX swap far leg):**

At $T = 1$:
- Receive $K N_f$ USD
- Pay $N_f$ EUR

This is equivalent to being **short EUR forward** at strike $K$.

Let $N_f = 10{,}000{,}000$ EUR and take $K = F_{\text{mkt}} = 1.1150$.

**PV in USD:**
$$\text{PV}_0 = K N_f P_d(0,1) - X(0) N_f P_f(0,1)$$

**Compute:**
- $K N_f P_d = 1.1150 \times 10{,}000{,}000 \times 0.98 = 10{,}927{,}000$
- $X(0) N_f P_f = 1.10 \times 10{,}000{,}000 \times 0.99 = 10{,}890{,}000$

So:
$$\text{PV}_0 = 10{,}927{,}000 - 10{,}890{,}000 = 37{,}000 \text{ USD}$$

**Equivalently:**
$$\text{PV}_0 = N_f P_d(0,1) (K - F^\star) = 10{,}000{,}000 \times 0.98 \times (1.1150 - 1.11122449) = 37{,}000 \text{ USD}$$

---

### Example F: Cross-Currency Swap Cashflow Table — 2-Period Toy XCCY with Consistent FX Conversion

**Inputs:**

| Parameter | Value |
|-----------|-------|
| Spot | $X(0) = 1.10$ USD/EUR |
| Payment dates | $t_1 = 0.5$, $t_2 = 1.0$; accruals $\tau_1 = \tau_2 = 0.5$ |
| Domestic (USD) discount factors | $P_d(0, 0.5) = 0.9900$, $P_d(0, 1.0) = 0.9800$ |
| Foreign (EUR) discount factors | $P_f(0, 0.5) = 0.9950$, $P_f(0, 1.0) = 0.9900$ |
| Domestic projection curve (toy) | $P_d^{(L)}(0, 0.5) = 0.9890$, $P_d^{(L)}(0, 1.0) = 0.9775$ |
| Foreign projection curve (toy) | $P_f^{(L)}(0, 0.5) = 0.9940$, $P_f^{(L)}(0, 1.0) = 0.9880$ |
| Notionals | $N_d = 100{,}000{,}000$ USD, $N_f = N_d / X(0) = 90{,}909{,}090.9091$ EUR |

**Step 1 — Compute forward floating rates from projection curves:**

Using $L = \frac{1}{\tau}\left(\frac{P^{(L)}(0, t_i)}{P^{(L)}(0, t_{i+1})} - 1\right)$:

*USD forwards:*
$$L_d(0; 0, 0.5) = \frac{1}{0.5}\left(\frac{1}{0.9890} - 1\right) = 0.0222446916$$

$$L_d(0; 0.5, 1.0) = \frac{1}{0.5}\left(\frac{0.9890}{0.9775} - 1\right) = 0.0235294118$$

*EUR forwards:*
$$L_f(0; 0, 0.5) = \frac{1}{0.5}\left(\frac{1}{0.9940} - 1\right) = 0.0120724346$$

$$L_f(0; 0.5, 1.0) = \frac{1}{0.5}\left(\frac{0.9940}{0.9880} - 1\right) = 0.0121457490$$

**Step 2 — Cashflow table (receive USD float, pay EUR float + basis $b$; basis solved in Example G):**

| Date | USD leg (receive) | EUR leg (pay) |
|------|-------------------|---------------|
| 0 | Exchange notionals: pay 100,000,000 USD, receive 90,909,090.9091 EUR | (same exchange, opposite sign) |
| 0.5 | $N_d \times L_d(0;0,0.5) \times 0.5 = 1{,}112{,}234.58$ USD | $N_f \times (L_f(0;0,0.5) + b) \times 0.5$ EUR |
| 1.0 | $N_d \times L_d(0;0.5,1.0) \times 0.5 = 1{,}176{,}470.59$ USD | $N_f \times (L_f(0;0.5,1.0) + b) \times 0.5$ EUR |
| 1.0 | Receive principal 100,000,000 USD | Pay principal 90,909,090.9091 EUR |

**Step 3 — FX-forward consistency check (convert foreign CFs using forwards):**

Compute forwards from discount factors (CIP-DF):
$$F(0,t) = X(0) \frac{P_f(0,t)}{P_d(0,t)}$$

$$F(0, 0.5) = 1.10 \times \frac{0.9950}{0.9900} = 1.10555556$$

$$F(0, 1.0) = 1.10 \times \frac{0.9900}{0.9800} = 1.11122449$$

Then for a foreign cashflow $C_f(t)$ EUR at $t$, domestic PV via forwards is:
$$\text{PV}^{(d)}(C_f(t)) = P_d(0,t) F(0,t) C_f(t)$$

which equals $X(0) P_f(0,t) C_f(t)$ if CIP holds (shown in Section 2.5).

---

### Example G: Par Basis Spread from Curves — Solve for $b_{par}$

We use the same 2-period swap from Example F.

**Step 1 — PV of USD leg in USD:**

- PV of coupon at 0.5: $1{,}112{,}234.58 \times P_d(0, 0.5) = 1{,}112{,}234.58 \times 0.99 = 1{,}101{,}112.23$
- PV of coupon at 1.0: $1{,}176{,}470.59 \times 0.98 = 1{,}152{,}941.18$
- PV of principal: $100{,}000{,}000 \times 0.98 = 98{,}000{,}000$

**Total:**
$$\text{PV}_d = 100{,}254{,}053.41 \text{ USD}$$

**Step 2 — PV of EUR leg without basis in EUR:**

*Coupon amounts (no basis):*
$$C_{f,0.5} = N_f L_f(0;0,0.5) \cdot 0.5 = 548{,}747.03 \text{ EUR}$$
$$C_{f,1.0} = N_f L_f(0;0.5,1.0) \cdot 0.5 = 552{,}079.50 \text{ EUR}$$

*Discount:*
- PV(coupon 0.5): $548{,}747.03 \times 0.9950 = 546{,}003.29$
- PV(coupon 1.0): $552{,}079.50 \times 0.9900 = 546{,}558.70$
- PV(principal): $90{,}909{,}090.9091 \times 0.9900 = 90{,}000{,}000$

**Total (no basis):**
$$\text{PV}_{f,0} = 91{,}092{,}561.9969 \text{ EUR}$$

**Converted to USD:** $X(0) \text{PV}_{f,0} = 1.10 \times 91{,}092{,}561.9969 = 100{,}201{,}818.20$ USD

**Step 3 — Add basis PV term and solve:**

Basis adds $b \tau_i N_f$ at each payment date. PV of basis coupons (in EUR):
$$\text{PV}_{\text{basis}}^{(f)} = N_f b \sum_{k=1}^{2} \tau_k P_f(0, t_k) = N_f b (0.5 \cdot 0.9950 + 0.5 \cdot 0.9900) = N_f b \cdot 0.9925$$

Compute the "basis annuity":
$$A_f = N_f \cdot 0.9925 = 90{,}227{,}272.7273$$

**Par condition** for receive-USD/pay-EUR+$b$:
$$0 = \text{PV}_d - X(0)(\text{PV}_{f,0} + A_f b) \Rightarrow b_{par} = \frac{\text{PV}_d / X(0) - \text{PV}_{f,0}}{A_f}$$

**Compute:**
- $\text{PV}_d / X(0) = 100{,}254{,}053.41 / 1.10 = 91{,}140{,}048.5555$ EUR
- Numerator $= 91{,}140{,}048.5555 - 91{,}092{,}561.9969 = 47{,}486.5586$ EUR
- Divide by $A_f = 90{,}227{,}272.7273$:

$$b_{par} = 0.00052629939 \approx 5.263 \text{ bp}$$

**Dependency takeaway:** Par basis depends on $X(0)$, both discount curves $P_d$, $P_f$, and projection curves (through the $L$ used in coupon amounts).

---

### Example H: Basis Shock Sensitivity — Widen Basis by +5bp and Compute PV Change

Start at par $b = b_{par} \Rightarrow \text{PV} = 0$.

**Shock:** $\Delta b = +5$ bp $= +0.0005$

**PV change:**
$$\Delta \text{PV} = -X(0) N_f \Delta b \sum_k \tau_k P_f(0, t_{k+1}) = -1.10 \times 90{,}909{,}090.9091 \times 0.0005 \times 0.9925 = -49{,}625 \text{ USD}$$

**Basis DV01 (per 1bp):**
$$\text{Basis DV01} = \frac{-49{,}625}{5} = -9{,}925 \text{ USD/bp}$$

**Interpretation:** Paying the basis leg makes you short basis (PV falls when basis widens).

---

### Example I: FX Delta of XCCY — Bump Spot FX by +1% and Compute PV Change

Start at par (PV = 0 at $X(0) = 1.10$). Hold curves and basis fixed.

**New spot:** $X'(0) = 1.10 \times 1.01 = 1.111$

At par, $\text{PV}_d = X(0) \text{PV}_f^{(f)} \Rightarrow \text{PV}_f^{(f)} = \text{PV}_d / X(0) = 91{,}140{,}048.5555$ EUR

**PV after spot bump:**
$$\text{PV}' = \text{PV}_d - X'(0) \text{PV}_f^{(f)} = \text{PV}_d - 1.111 \times 91{,}140{,}048.5555 = -1{,}002{,}540.53 \text{ USD}$$

So a +1% spot move produced **-\$1.003mm P&L** for this receive-USD/pay-EUR position.

**FX delta (holding curves fixed):**
$$\frac{\partial \text{PV}}{\partial X(0)} = -\text{PV}_f^{(f)} \approx -91.14 \text{ million EUR}$$

---

### Example J: Domestic vs Foreign PV01 — Bump Curves +1bp Separately and Compare

We use the par swap (PV = 0 at base).

**Bump definition (toy, discount-factor-based):**
- Define simple zero rate at maturity $t$ by $P(0,t) = \frac{1}{1 + r(t) t}$
- Apply a parallel +1bp bump: $r'(t) = r(t) + 0.0001$
- Recompute $P'(0,t) = \frac{1}{1 + r'(t) t}$

**We hold:**
- Projection curves fixed
- Spot FX fixed
- Basis fixed at $b_{par}$

#### J1) Domestic Discount Curve +1bp (Foreign Unchanged)

**Compute bumped domestic DFs:**
- $P_d'(0, 0.5) = 0.9899509974$
- $P_d'(0, 1.0) = 0.9799039694$

**Recompute USD leg PV:**
- PV(coupon 0.5): $1{,}112{,}234.58 \times 0.9899509974 = 1{,}101{,}057.73$
- PV(coupon 1.0): $1{,}176{,}470.59 \times 0.9799039694 = 1{,}152{,}828.20$
- PV(principal): $100{,}000{,}000 \times 0.9799039694 = 97{,}990{,}396.94$

**Total:**
$$\text{PV}_d' = 100{,}244{,}282.87 \text{ USD}$$

Since foreign leg unchanged and base PV=0, new PV:
$$\Delta \text{PV} = \text{PV}_d' - \text{PV}_d = -9{,}770.54 \text{ USD}$$

**Domestic PV01 $\approx -9{,}770.54$ USD per +1bp**

#### J2) Foreign Discount Curve +1bp (Domestic Unchanged)

**Compute bumped foreign DFs:**
- $P_f'(0, 0.5) = 0.9949505012$
- $P_f'(0, 1.0) = 0.9899019997$

**Recompute EUR leg PV (including basis coupons at $b_{par}$):**

*Basis coupon amount each period:*
$$N_f b_{par} \tau = 90{,}909{,}090.9091 \times 0.00052629939 \times 0.5 = 23{,}922.70 \text{ EUR}$$

*Total coupon amounts:*
- at 0.5: $548{,}747.03 + 23{,}922.70 = 572{,}669.73$
- at 1.0: $552{,}079.50 + 23{,}922.70 = 576{,}002.20$

*Discount with bumped DFs:*
- PV(coupon 0.5): $572{,}669.73 \times 0.9949505012 = 569{,}778.03$
- PV(coupon 1.0): $576{,}002.20 \times 0.9899019997 = 570{,}185.73$
- PV(principal): $90{,}909{,}090.9091 \times 0.9899019997 = 89{,}991{,}090.88$

**Total:**
$$\text{PV}_f' = 91{,}131{,}054.64 \text{ EUR}$$

**Change in foreign PV (EUR):**
$$\Delta \text{PV}_f^{(f)} = 91{,}131{,}054.64 - 91{,}140{,}048.56 = -8{,}993.91 \text{ EUR}$$

**Convert impact to USD PV:**
$$\Delta \text{PV} = -X(0) \Delta \text{PV}_f^{(f)} = -1.10 \times (-8{,}993.91) = +9{,}893.30 \text{ USD}$$

**Foreign PV01 $\approx +9{,}893.30$ USD per +1bp** (for this receive-USD/pay-EUR position)

**Interpretation:** Raising foreign rates reduces the PV of what you owe in EUR, which helps you (PV increases).

---

### Example K: Hedging a Foreign Floating Exposure — Transform EUR Floating Asset into Mostly USD Floating Exposure

**Starting position (asset):** Receive a EUR floating cashflow stream on $N_f = 90.9091$m EUR with principal at $T = 1$ (same schedule as above), valued in USD.

**PV of EUR floater (no basis) in EUR:**
$$\text{PV}_{f,0} = 91{,}092{,}561.9969 \text{ EUR}$$
(from Example G, basis = 0)

**USD PV:**
$$\text{PV}_{\text{asset}} = X(0) \text{PV}_{f,0} = 1.10 \times 91{,}092{,}561.9969 = 100{,}201{,}818.20 \text{ USD}$$

**Hedge instrument:** Enter a par cross-currency basis swap (from Example G) that:
- Receives USD float + principal on $N_d = 100$m USD
- Pays EUR float + $b_{par}$ + principal on $N_f$
- With $b_{par} = 5.263$ bp so the swap PV = 0 at inception

**Exposure summary (toy, held-fixed definitions as above):**

| Risk Factor | Before Hedge (EUR floating asset) | Swap Hedge (receive USD, pay EUR+$b_{par}$) | After Hedge (asset + swap) |
|-------------|-----------------------------------|---------------------------------------------|---------------------------|
| PV | 100,201,818.20 USD | 0 (par by construction) | 100,201,818.20 USD |
| FX delta | $+\text{PV}_{f,0} = +91{,}092{,}562$ EUR | $-\text{PV}_f^{(f)}(b_{par}) = -91{,}140{,}049$ EUR | $-47{,}487$ EUR |
| Foreign PV01 | $\approx -9{,}889.42$ USD/bp | $+9{,}893.30$ USD/bp | $+3.88$ USD/bp $\approx 0$ |
| Domestic PV01 | 0 | $-9{,}770.54$ USD/bp | $-9{,}770.54$ USD/bp |
| Basis DV01 | 0 | $-9{,}925$ USD/bp | $-9{,}925$ USD/bp |

**Interpretation:** The hedge largely removes foreign rate and FX exposure, transforming the position into predominantly USD floating exposure plus cross-currency basis exposure.

---

### Example L: Scenario P&L Explain — Decompose P&L for a Toy Par XCCY Position

**Position:** The par swap from Examples G-J (receive USD float, pay EUR float + basis), PV = 0 at start.

We run three separate "one-factor" scenarios, holding other inputs fixed (as defined in Section 3C).

#### Scenario 1: Spot FX Move

Spot increases by +2%: $X(0)$ from 1.10 to 1.122, so $\Delta X = 0.022$

FX delta $\approx -91{,}140{,}048.56$ EUR

**P&L:**
$$\Delta \text{PV}_{\text{FX}} \approx -91{,}140{,}048.56 \times 0.022 = -2{,}005{,}081.07 \text{ USD}$$

**Operational meaning:** You are short foreign PV; stronger foreign currency makes the foreign leg more expensive in USD terms.

#### Scenario 2: Domestic Curve Shift

Domestic discount curve +10bp parallel shift (toy PV01 linearization)

Domestic PV01 $\approx -9{,}770.54$ USD/bp

**P&L:**
$$\Delta \text{PV}_d \approx -9{,}770.54 \times 10 = -97{,}705.38 \text{ USD}$$

**Operational meaning:** You are long USD cashflows; higher USD discounting lowers PV.

#### Scenario 3: Basis Widening

Basis widens by +20bp

Basis DV01 $\approx -9{,}925$ USD/bp

**P&L:**
$$\Delta \text{PV}_{\text{basis}} \approx -9{,}925 \times 20 = -198{,}500 \text{ USD}$$

**Operational meaning:** Paying basis makes you short the basis spread.

#### Combined (Linearized) P&L (if moves are small and independent):

$$\Delta \text{PV} \approx \Delta \text{PV}_{\text{FX}} + \Delta \text{PV}_d + \Delta \text{PV}_{\text{basis}}$$

**Note:** In real desks, cross-effects (recalibration, FX-forward consistency, collateral discounting) may matter; those belong in a fuller curve/XVA treatment.

---

## Practical Notes

### Quoting and Convention Ambiguity Checklist

| Item | What to Verify |
|------|----------------|
| **FX quote direction and forward points** | Confirm whether your systems quote domestic per foreign or the inverse; Hull notes some currencies are quoted differently (e.g., "1 USD = x JPY" vs "1 JPY = x USD"). Confirm whether forwards are quoted as outright $F$ or as "forward points" added to spot (I'm not sure this is covered in the provided excerpts). |
| **Which leg gets basis spread and sign convention** | The reference describes "plus or minus a spread" and uses an example where the non-USD leg (JPY) has $+e$. I'm not sure your market convention: some desks quote basis as added to the USD leg vs non-USD leg; confirm with confirmations/broker screens. |
| **Whether notionals exchange at inception/maturity** | Both the currency swap description (principals exchanged at start/end) and the basis swap description (notionals exchanged at inception/maturity, ratio set to spot) indicate notional exchanges are part of the canonical structure in these texts. Variants exist in practice; confirm product standard if needed. |
| **Collateral currency and discounting curve choice (preview)** | OIS/collateral discounting is motivated in the reference; the collateral rate can drive discounting rather than Libor proxies. I'm not sure how your desk's CSA (collateral currency/rate, thresholds, frequency) translates into a specific multi-currency discount curve without additional documentation. |

### Common Pitfalls

1. **Mixing up projection vs discount curves in each currency**
   - The reference explicitly separates discounting vs Libor projection and introduces pseudo-discount curves for forecasting vs outright discounting

2. **Inconsistent FX conversion timing (spot vs forward) in PV**
   - If you use forwards, ensure they are consistent with the curves (or explicitly accept inconsistency for a "risk decomposition" calculation)

3. **Treating basis as "just a spread" without recognizing embedded curve dependencies**
   - The reference's arbitrage scheme shows inconsistent basis/curves creates arbitrage opportunities even though trades are "costless"

4. **Confusing FX swap with XCCY swap (funding vs long-dated transformation)**
   - FX swaps are short-dated funding-style instruments; XCCY swaps are long-dated currency + rate-index transformations (often with principal exchanges)

### Implementation Pitfalls

1. **Date alignment across calendars**
   - I'm not sure the provided excerpts specify calendar/rolling conventions for these products; keep a robust date engine and confirm for your currency pair

2. **Interpolation artifacts creating inconsistent forward FX strips**
   - Ensure the implied forward FX curve $F(0,t)$ is consistent with discount curves and any basis adjustments used

3. **Bump-and-rebuild consistency for basis risk**
   - Decide whether "basis bump" holds curves fixed or rebuilds curves to market; document which you are doing

### Verification Tests

| Test | What to Check |
|------|---------------|
| **PV(spot+forward) = 0 at the no-arbitrage forward** | Verify $K^\star = X(0) \frac{P_f}{P_d}$ produces PV = 0 for an FX forward (Example B) |
| **Unit checks (currency units, DF units)** | Every PV term must be in the same currency; $P$ is dimensionless; $X$ carries currency units |
| **Small-bump stability for PV01/basis DV01** | Compare PV changes for 1bp vs 2bp bumps (linearity check) |
| **Consistency between FX forwards and curves** | If you value foreign cashflows using domestic forwards, ensure $P_d F = X(0) P_f$ holds when expected |

---

## Summary & Recall

### 10-Bullet Executive Summary

1. **FX forwards link spot FX and interest rates via parity:** $F = X \frac{P_f}{P_d}$ (discount-factor form)

2. **Hull's parity form:** $F_0 = S_0 e^{(r - r_f)T}$ (continuous-compounding statement)

3. **An FX swap can be viewed as spot + opposite-direction forward**, i.e., two notional exchanges at different dates

4. **FX swaps implement funding:** borrow one currency, lend the other, locked by FX

5. **Currency swaps exchange principals at start and end;** floating-for-floating swaps exchange floating legs (possibly with spreads)

6. **Cross-currency basis swaps** exchange floating Libor legs across currencies plus/minus a spread; notional ratio set at spot at inception

7. **Modern valuation depends on discount curves for PV and projection curves for floating rates;** do not conflate them

8. **Cross-currency basis must be fit consistently;** otherwise costless swap chains imply arbitrage (source's scheme)

9. **Key risks:** FX delta, domestic PV01, foreign PV01, basis DV01—each depends on what you hold fixed

10. **Collateralization can change discounting** (often OIS-based); multi-currency collateral effects require CSA-specific details

---

### Cheat Sheet: CIP Identities, FX Swap PV, XCCY PV Equation Structure, Exposure Decomposition

**CIP (discount-factor):**
$$\boxed{F(0,T) = X(0) \frac{P_f(0,T)}{P_d(0,T)}}$$

**FX forward PV (receive 1 foreign, pay $K$ domestic at $T$):**
$$\boxed{\text{PV}_0^{(d)} = X(0) P_f(0,T) - K P_d(0,T)}$$

**FX swap PV (far leg receive $KN_f$ domestic, pay $N_f$ foreign):**
$$\boxed{\text{PV}_0^{(d)} = K N_f P_d(0,T) - X(0) N_f P_f(0,T) = N_f P_d(0,T)(K - F^\star(0,T))}$$

**Cross-currency basis swap PV skeleton (receive domestic float, pay foreign float + basis):**
$$\boxed{\text{PV}^{(d)} = \text{PV}_d^{(d)} - X(0) \text{PV}_f^{(f)}(b)}$$

with $\text{PV}_f^{(f)}(b)$ linear in $b$ via $\sum \tau_i P_f(0, t_i)$ (see XCCY-PV)

**Risk decomposition (held-fixed):**
- FX delta: $\frac{\partial \text{PV}}{\partial X(0)} = -\text{PV}_f^{(f)}$
- Basis DV01: $\frac{\partial \text{PV}}{\partial b} \approx -X(0) N_f \sum \tau_i P_f$
- PV01s: recompute PV under curve bump with explicit "held fixed" rule

---

### 30 Flashcards (Q/A)

1. **Q:** What is the chapter's default FX quote direction?
   **A:** Domestic currency per 1 unit of foreign currency (e.g., USD/EUR).

2. **Q:** Write CIP in discount-factor form.
   **A:** $F(0,T) = X(0) \frac{P_f(0,T)}{P_d(0,T)}$

3. **Q:** Write Hull's continuous-compounding forward FX formula.
   **A:** $F_0 = S_0 e^{(r - r_f)T}$

4. **Q:** What market detail can flip your interpretation of $X$?
   **A:** Quote direction differs by currency pair; some are quoted as "1 USD = x JPY," etc.

5. **Q:** Domestic PV of FX forward receiving 1 foreign and paying $K$ domestic?
   **A:** $X(0) P_f(0,T) - K P_d(0,T)$

6. **Q:** When does an FX forward have zero PV at inception?
   **A:** When $K = F(0,T) = X(0) \frac{P_f}{P_d}$

7. **Q:** How can you view an FX swap structurally?
   **A:** Spot FX trade + opposite-direction FX forward (two notional exchanges at different dates)

8. **Q:** Funding interpretation of "spot buy foreign + forward sell foreign"?
   **A:** Borrow foreign, lend domestic

9. **Q:** What is a currency swap in Hull's description?
   **A:** Two principals in two currencies are exchanged at the beginning and end

10. **Q:** Define a floating-for-floating currency swap.
    **A:** Floating rate on each currency principal (possibly with spread added) exchanged between parties

11. **Q:** How does Hull suggest valuing floating-for-floating currency swaps?
    **A:** Assume forward rates realized, discount each currency's cashflows on its zero curve, convert PVs using current exchange rate

12. **Q:** What is a cross-currency basis swap in the interest-rate modeling text?
    **A:** Exchange floating Libor payments in one currency for floating Libor payments in another plus/minus a spread; exchange notionals at inception and maturity

13. **Q:** What does the text say about FX forwards' maturity liquidity?
    **A:** Interbank FX forwards are rarely liquid beyond ~1 year

14. **Q:** Why do basis swaps matter for curve consistency?
    **A:** If you don't fit them, you can create arbitrageable inconsistencies (zero-cost swap chains)

15. **Q:** What is a "CRX yield spread" $s(t)$ in the text?
    **A:** A yield-space spread between pseudo-discount $P^{(L)}$ and discount $P$: $P^{(L)}(t) = P(t) e^{-s(t) t}$

16. **Q:** What does $P^{(L)}$ represent conceptually?
    **A:** A projection (pseudo-discount) curve used to forecast Libor-like rates, distinct from discounting curve $P$

17. **Q:** How do you compute forward Libor from $P^{(L)}$?
    **A:** $L = \frac{1}{\tau}\left(\frac{P^{(L)}(t_i)}{P^{(L)}(t_{i+1})} - 1\right)$

18. **Q:** What is Assumption 6.5.1 used for in the reference's basis-swap PV?
    **A:** It sets USD pseudo-discount curve equal to USD discount curve (simplifying USD floater PV to 1)

19. **Q:** What objects enter a cross-currency basis swap PV?
    **A:** Domestic discount curve, foreign discount curve, projection curves (to get $L$), spot FX, and basis spread (additive)

20. **Q:** What is "basis DV01"?
    **A:** PV change per 1bp change in the cross-currency basis spread (definition depends on what is held fixed)

21. **Q:** If $\text{PV} = \text{PV}_d - X \text{PV}_f$, what is FX delta?
    **A:** $\frac{\partial \text{PV}}{\partial X} = -\text{PV}_f$ (foreign PV in foreign units)

22. **Q:** Why is converting all foreign cashflows at spot generally wrong?
    **A:** It ignores timing/hedging; forwards and discounting must be consistent (CIP) unless explicitly modeling deviations

23. **Q:** How do basis swaps relate to FX forwards per the text?
    **A:** A one-period cross-currency basis swap is identical to an FX forward contract

24. **Q:** What does the text say about collateralization and discounting?
    **A:** In collateralized inter-dealer markets, discounting ties to collateral/OIS; Libor is not a risk-free proxy

25. **Q:** What is the main "what curve does what?" rule?
    **A:** Discount curves discount cashflows; projection curves generate floating rates

26. **Q:** What is a common arbitrage check in cross-currency?
    **A:** Price same USD cashflows via (i) USD discount + spot conversion vs (ii) swap chain using basis swap; discrepancies imply arbitrage

27. **Q:** What does "par basis" mean?
    **A:** Basis spread that makes the basis swap PV zero at inception

28. **Q:** What is DV01 in Tuckman's expression?
    **A:** $\text{DV01} = -D \times P \times 0.0001$

29. **Q:** Why must valuation procedures make new deals have zero value?
    **A:** Otherwise traders can arbitrage inconsistent system pricing; Hull emphasizes discount rates may be adjusted to ensure new trades have zero value

30. **Q:** What's the single most important convention to confirm before pricing/risking XCCY?
    **A:** Which leg gets the basis (and its sign) plus the FX quote direction

---

## Mini Problem Set (16 Questions)

*Increasing difficulty. Solution sketches provided for questions 1-8 only.*

### Questions with Solution Sketches

**Q1:** Given $X(0)$, $P_d(0,T)$, $P_f(0,T)$, compute the no-arbitrage forward $F(0,T)$.

*Sketch:* Use $F = X \frac{P_f}{P_d}$. Unit check.

---

**Q2:** You have an FX forward to receive 1 foreign and pay $K$ domestic at $T$. Derive PV in domestic currency using discount factors.

*Sketch:* $\text{PV} = X(0) P_f - K P_d$. Interpret as long foreign ZCB, short domestic ZCB.

---

**Q3:** Show algebraically that valuing a foreign cashflow $C_f(T)$ via forward FX and domestic discounting equals spot conversion and foreign discounting under CIP.

*Sketch:* Use $P_d F = X(0) P_f$ from CIP-DF.

---

**Q4:** In the interest-rate modeling reference's arbitrage scheme, explain why inconsistent basis-swap inputs imply arbitrage.

*Sketch:* Trades 1-3 are costless; they transform USD fixed into JPY fixed. If PV differs from direct USD discount + spot conversion, you have a discrepancy with zero-cost strategy → arbitrage.

---

**Q5:** In a floating-floating currency swap, list the market objects needed to produce PV in USD.

*Sketch:* USD discount curve, GBP/EUR discount curve, spot FX, forward rates (from projection curves or implied forwards), and basis spread if present.

---

**Q6:** For a 2-period foreign leg paying $L_f + b$, show that PV is affine (linear) in $b$.

*Sketch:* Expand $\text{PV} = \text{PV}(\text{with } b=0) + b N_f \sum \tau_i P_f(0, t_i)$.

---

**Q7:** Define FX delta for $\text{PV} = \text{PV}_d - X \text{PV}_f$.

*Sketch:* Differentiate: $\frac{\partial \text{PV}}{\partial X} = -\text{PV}_f$.

---

**Q8:** Explain why "domestic PV01" for a cross-currency swap is not uniquely defined unless you specify what is held fixed.

*Sketch:* Because curves, FX forwards, and basis can be linked by arbitrage; bumping one input may require re-implying others to keep consistency. State your chosen bump rule.

---

### Questions without Solutions (for Practice)

**Q9:** Compute par basis $b_{par}$ for a 3-period swap given discount factors and projection curves (numeric).

**Q10:** Show how the sign of basis DV01 flips when you switch from "pay basis" to "receive basis."

**Q11:** If FX forwards beyond 1Y are illiquid, explain how cross-currency basis swaps help tie down long-dated cross-currency relationships.

**Q12:** For a given XCCY swap, propose a hedging set to neutralize FX delta and domestic PV01, leaving basis risk.

**Q13:** Explain how collateralization can alter the "right" discount curve for a cross-currency swap, and what additional data you'd need to implement it.

**Q14:** Design a bump scheme that measures "CIP-consistent" domestic PV01 (i.e., bump domestic curve and adjust forward FX accordingly).

**Q15:** Consider a trade that is a strip of foreign cashflows. Compare valuing via (i) foreign discount + spot and (ii) domestic discount + forwards. Under what assumptions are they identical?

**Q16:** Create a P&L explain template that separates moves into FX spot, domestic curve, foreign curve, and basis, and state the limitations.

---

## Source Map

### (A) Verified Facts — Cite Specific Sources

| Fact | Source |
|------|--------|
| FX forward parity $F_0 = S_0 e^{(r-r_f)T}$ | Hull, *Options, Futures, and Other Derivatives*, Ch 5 |
| Forward FX linked to discount factors | Andersen & Piterbarg, *Interest Rate Modeling*, Vol 1 |
| Currency swap structure (principals exchanged at start/end) | Hull, *Options, Futures, and Other Derivatives*, Ch 7 |
| Floating-for-floating swap valuation procedure | Hull, *Options, Futures, and Other Derivatives*, Ch 7 |
| Cross-currency basis swap definition and mechanics | Andersen & Piterbarg, *Interest Rate Modeling*, Vol 1, Ch 6 |
| Forward rate from projection curve formula | Andersen & Piterbarg, *Interest Rate Modeling*, Vol 1, Ch 6 |
| One-period basis swap = FX forward | Andersen & Piterbarg, *Interest Rate Modeling*, Vol 1 |
| FX forwards rarely liquid beyond 1 year | Andersen & Piterbarg, *Interest Rate Modeling*, Vol 1 |
| Collateralized discounting ties to collateral/OIS | Andersen & Piterbarg, *Interest Rate Modeling*, Vol 1 |
| DV01 formula | Tuckman, *Fixed Income Securities*, Ch 5-6 |
| XCCY-PV formula (USD/JPY example) | Andersen & Piterbarg, *Interest Rate Modeling*, Vol 1, Ch 6 |

### (B) Reasoned Inference — Note Derivation Logic

| Inference | Derivation |
|-----------|------------|
| CIP in discount-factor form at $t=0$ | Algebraic specialization of Andersen & Piterbarg forward FX formula |
| FX forward PV formula | Replication argument using ZCBs in each currency |
| Spot conversion = forward conversion under CIP | Algebra substituting CIP-DF |
| FX delta = $-\text{PV}_f^{(f)}$ | Partial derivative of PV formula |
| Basis DV01 linear structure | Differentiation of XCCY-PV with respect to additive basis |
| Par basis formula | Setting $\text{PV} = 0$ and solving |

### (C) Speculation — Flag Uncertainties

| Uncertainty | What Would Be Needed |
|-------------|----------------------|
| Canonical "FX swap" legal definition | Market/documentation definition, settlement conventions |
| Universal sign convention for basis | Market quoting convention for specific currency pair |
| XCCY variants (resettable notionals, no-initial-exchange) | Product standard and specific section |
| Multi-currency collateral discounting formula | CSA collateral currency, remuneration rate, framework section |
| "FX swap implied funding spread" metric | Money-market conventions, OIS/CSA basis, unsecured vs collateralized |
| Calendar/rolling conventions for XCCY products | Product documentation for currency pair |
| Forward points quoting convention | Market convention (not confirmed in excerpts) |
