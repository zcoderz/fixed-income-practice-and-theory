# Chapter 21: Cross-Currency Curves — CIP, FX Forwards, and Cross-Currency Basis (as Curve Constraints)

---

## Fact Classification

### (A) Verified Facts (Source-Backed)
- CIP relates spot FX, forward FX, and interest-rate discounting across two currencies (Hull Ch 5)
- Forward FX rate in discount-factor form: $F(0,T) = X(0) \frac{P_f(0,T)}{P_d(0,T)}$ (Andersen Vol 1)
- Forward FX rate in continuous-compounded rate form: $F_0 = S_0 e^{(r_d - r_f)T}$ (Hull Ch 5)
- Multi-curve framework separates discount curve $P(\cdot)$ from index/pseudo-discount curve $P^{(L)}(\cdot)$ (Andersen Vol 1 Ch 6)
- Cross-currency basis swaps exchange floating LIBOR in two currencies plus/minus a spread, with notional exchanges at inception and maturity at spot FX ratio (Andersen Vol 1)
- A one-period cross-currency basis swap is identical to an FX forward (Andersen Vol 1)
- Building curves independently from local swaps can violate cross-currency no-arbitrage constraints (Andersen Vol 1 Ch 6)
- DV01 is defined as the price change for a 1bp yield change: $\text{DV01} = -\frac{\partial P}{\partial y} \times 0.0001$ (Tuckman Ch 5)

### (B) Reasoned Inference (Derived from A)
- FX-implied foreign discount factor: $P_f(0,T) = \frac{F(0,T)}{X(0)} P_d(0,T)$ (algebra from CIP-DF)
- Cross-currency yield spread formula: $s(t) = -\frac{1}{t} \ln\left(\frac{P^{(L)}(t)}{P(t)}\right)$ (algebra from yield spread definition)
- The ratio $P_f/P_d$ represents relative growth of foreign and domestic money-market numeraires
- Basis swap par conditions constrain the relationship between projection and discount curves

### (C) Speculation (Clearly Labeled; Minimal)
- I'm not sure about exact spot/settlement conventions as they are currency-pair specific (spot lag, holidays, business day rules, fixing conventions)
- I'm not sure about exact fails-charge formulas — sources discuss fails conceptually but don't specify penalty mechanics
- I'm not sure about basis swap quoting conventions (which leg gets the spread, sign conventions) as this is market-convention specific
- I'm not sure about building consistent calendars across currencies — book does not give full operational calendars

---

## Conventions & Notation

| Symbol | Meaning | Units/Notes |
|--------|---------|-------------|
| $d$ | Domestic (reporting) currency | — |
| $f$ | Foreign currency | — |
| $t$ | Valuation time | years |
| $T$ | Maturity | years |
| $\tau$ | Year fraction / accrual factor | years |
| $P_d(0,T)$ | Domestic discount factor for maturity $T$ | dimensionless |
| $P_f(0,T)$ | Foreign discount factor for maturity $T$ | dimensionless |
| $X(0)$ | Spot FX: domestic price of 1 unit foreign | $d/f$ |
| $F(0,T)$ or $X_T(0)$ | Forward FX rate for delivery at $T$ | $d/f$ (same as spot) |
| $Y(0,T)$ | Inverse-quoted forward FX: $1/F(0,T)$ | $f/d$ |
| $r_d, r_f$ | Continuous-compounded zero rates | 1/year |
| $L(0,T_i,T_{i+1})$ | Simple forward LIBOR over $[T_i, T_{i+1}]$ | 1/year |
| $P^{(L)}(0,T)$ | Index/pseudo-discount curve for LIBOR | dimensionless |
| $s(t)$ | Cross-currency (CRX) yield spread | 1/year (often quoted in bp) |
| $e$ | Cross-currency basis swap spread | 1/year (often quoted in bp) |

### Defaults Used in Examples
- **Quote direction**: $X(0)$ is domestic per 1 foreign ($d/f$)
- **Compounding**: Continuous unless otherwise stated
- **Rounding**: Values rounded to 6 decimals (rates) or 6–7 significant digits (DFs/FX)

### Important Market Caveats
- FX markets often quote in the opposite direction (e.g., "units of currency per USD")
- Formulas must be inverted depending on quote direction
- Basis swap spread conventions (which leg, sign) vary by market — always confirm with desk

---

## Core Concepts

### 1) Covered Interest Parity (CIP)

**Formal Definition:**
CIP is the no-arbitrage relationship linking spot FX, forward FX, and interest rate discounting in two currencies, such that a fully FX-hedged (covered) investment yields the same return as a domestic investment. In rate form (continuous compounding):

$$\boxed{F(0,T) = S_0 \, e^{(r_d - r_f)T}}$$

where $S_0$ is spot FX quoted as domestic per foreign, $r_d$ and $r_f$ are risk-free rates in domestic and foreign currencies.

**Intuition:**
If the forward is "too high" relative to the interest differential, you can:
1. Borrow one currency, lend the other
2. Lock in the FX conversion with a forward

and generate an arbitrage profit (until prices move back into parity). Hull explicitly frames the forward formula as "interest rate parity."

**Trading / Risk / Portfolio Practice:**
- Used to sanity-check quoted FX forwards against the two money-market curves
- In curve building, CIP becomes an equation constraint tying the foreign discount curve to the domestic discount curve and the FX forward curve

---

### 2) FX Forward as a No-Arbitrage Instrument

**Formal Definition:**
An FX forward is a contract to exchange currencies at a fixed rate at time $T$. Its fair forward rate makes the contract's initial PV equal to zero (no-arbitrage).

**Intuition:**
It is (economically) a package: "own foreign cashflow at $T$" + "short domestic cashflow at $T$" with the exchange rate fixed in advance.

**Trading / Risk / Portfolio Practice:**
- FX forwards are used to hedge foreign currency cashflows and to synthesize foreign funding
- A strip of forwards can be treated as market quotes that constrain curve construction (especially short maturities)

---

### 3) Discount-Factor Form of CIP (Forward FX Rate via ZCB Replication)

**Formal Definition:**
In a domestic/foreign setup with spot $X(t)$ and discount factors $P_d(t,T)$, $P_f(t,T)$, the $T$-maturity forward FX rate is:

$$\boxed{X_T(t) = X(t) \frac{P_f(t,T)}{P_d(t,T)}}$$

The book motivates this by replicating a forward FX exposure using a foreign ZCB funded by a domestic ZCB.

**Intuition:**
The ratio $P_f/P_d$ is the relative growth of the foreign and domestic money-market numeraires over $[t,T]$ (in discount-factor form). Multiply by spot to get the forward.

**Trading / Risk / Portfolio Practice:**
- This is the practical "bridge" between a rates curve object (discount factors) and an FX object (forward curve)
- Any curve set used on a desk must satisfy this relationship (after applying correct market quoting conventions)

---

### 4) FX-Implied Curve

**Formal Definition:**
Given:
- A domestic discount curve $P_d(0,T)$
- Spot FX $X(0)$
- A set of FX forwards $F(0,T)$ (same quote direction as $X(0)$)

you can infer an implied foreign discount factor:

$$\boxed{P_f(0,T) = \frac{F(0,T)}{X(0)} P_d(0,T)}$$

(This is algebra from the discount-factor CIP formula.)

**Intuition:**
FX forwards "import" the foreign curve into the domestic curve: they tell you how many domestic discount units equal one unit of foreign discounting at $T$.

**Trading / Risk / Portfolio Practice:**
- Used for valuing and hedging hedged foreign assets
- Input into multi-currency curve bootstrapping

---

### 5) Cross-Currency Basis (Conceptual)

**Formal Definition:**
The multi-curve view defines a pseudo-discount (index) curve $P^{(L)}$ for LIBOR forwards and a discount curve $P$ for discounting. Their difference can be summarized as a yield spread:

$$\boxed{P^{(L)}(t) = P(t) e^{-s(t) t}}$$

where $s(t)$ is called the cross-currency (CRX) yield spread.

**Intuition:**
A nonzero $s(t)$ captures that "the curve implied by IBOR-style forwards" is not the same as "the curve used for discounting risk-free cashflows," and that this difference can matter across currencies (funding/credit/liquidity segmentation).

**Trading / Risk / Portfolio Practice:**
- Traders talk about "basis" when a currency's funding through FX markets is more/less expensive than what naïve CIP using a single curve would suggest
- Basis can "blow out" in stress (e.g., JPY late 1990s, and large positive basis in early 2008)

---

### 6) Cross-Currency Basis Swaps

**Formal Definition:**
A floating–floating cross-currency basis swap exchanges:
- Floating LIBOR in one currency
- For floating LIBOR in another currency
- Plus or minus a quoted spread

with notional exchanges at inception and maturity, where the notional ratio is normally set to the spot FX at trade inception. A one-period CRX basis swap is identical to an FX forward.

**Intuition:**
For maturities beyond where FX forwards are liquid (often beyond ~1 year), the basis swap market supplies the longer-dated "cross-currency constraint" information.

**Trading / Risk / Portfolio Practice:**
- Basis swaps are used to hedge FX funding risk of long-dated foreign cashflows
- Used to build consistent multi-currency curves

---

### 7) "Curve Constraints" Viewpoint

**Formal Definition:**
Multi-currency curve construction must respect arbitrage constraints coming from FX forwards and cross-currency basis swaps.

**Intuition:**
If you build USD and JPY curves independently from their local swaps, you may violate the FX forward relationship, creating cross-currency arbitrage opportunities.

**Trading / Risk / Portfolio Practice:**
- Consistent curve sets reduce "model arbitrage" and prevent systematic mispricing of cross-currency instruments

---

## Math and Derivations

### 2.1 Domestic Value of a Foreign Zero-Coupon Bond

From the domestic/foreign setup: a $T$-maturity foreign discount bond is worth $P_f(t,T)$ in foreign currency at time $t$. Its domestic price is spot FX times that amount:

$$\tilde{P}_d(t,T) = X(t) \, P_f(t,T)$$

where $X(t)$ is domestic per foreign.

**Unit check:**
- $X(t)$: $d/f$
- $P_f(t,T)$: dimensionless (foreign currency units per unit payoff)
- $\tilde{P}_d$: $d$ (domestic currency) ✓

---

### 2.2 Deriving the Forward FX Rate in Discount-Factor Form (CIP in DF Form)

**Replication idea** (as stated in the text): a forward FX position can be replicated by buying a foreign ZCB and selling a domestic ZCB, leading to the forward FX rate:

$$\boxed{X_T(t) = X(t) \frac{P_f(t,T)}{P_d(t,T)}} \tag{CIP-DF}$$

**Sanity check:** If $P_d = P_f$, then $X_T = X$ (no interest differential $\Rightarrow$ forward equals spot). ✓

---

### 2.3 CIP in Rate Form and Link to Discount Factors

Hull gives (with continuous compounding) the currency forward pricing identity:

$$\boxed{F_0 = S_0 e^{(r - r_f)T}} \tag{Hull-CIP}$$

If we define discount factors under continuous compounding $P(0,T) = e^{-rT}$, then:

$$\frac{P_f(0,T)}{P_d(0,T)} = \frac{e^{-r_f T}}{e^{-r_d T}} = e^{(r_d - r_f)T}$$

Plugging into (CIP-DF) recovers (Hull-CIP) with $S_0 = X(0)$.

**Unit check:** $r$ has units $1/\text{year}$; $rT$ is dimensionless; exponentials are dimensionless. ✓

---

### 2.4 Quote-Direction Variants (Spot and Forward)

The cross-currency curve construction section uses a specific example where:
- $X(0)$ is quoted in $/¥ terms (domestic per foreign)
- The FX forward deliverable is described by a rate $Y(T)$ in the inverse direction (foreign per domestic)

No-arbitrage requires:

$$\boxed{P_\$(T) = X(0) \, P_¥(T) \, Y(T) \quad \Rightarrow \quad P_¥(T) = \frac{P_\$(T)}{Y(T) X(0)}} \tag{6.38}$$

This is the same CIP relationship, but written using the inverse forward quote $Y(T)$.

**Key practical warning:** FX quote direction varies by market; Hull explicitly notes many spot rates are quoted as "units of currency per USD," which flips the algebra unless you invert the FX quote.

---

### 2.5 PV of an FX Forward as PV of Two Cashflows (Two-Currency Decomposition)

Consider a forward contract that at time $T$:
- Receives 1 unit of foreign currency $f$
- Pays $K$ units of domestic currency $d$

Time-0 domestic PV:

$$V_0 = \underbrace{X(0) P_f(0,T)}_{\text{PV of 1}f\text{ at }T\text{ in }d} - \underbrace{K P_d(0,T)}_{\text{PV of }K d\text{ at }T}$$

Setting $V_0 = 0$ gives:

$$K = X(0) \frac{P_f(0,T)}{P_d(0,T)} = F(0,T)$$

i.e., the CIP-implied forward FX rate (CIP-DF).

---

### 2.6 FX Forwards as Curve Constraints (Implied Curve Algebra)

From (CIP-DF) at $t=0$:

$$F(0,T) = X(0) \frac{P_f(0,T)}{P_d(0,T)} \quad \Rightarrow \quad \boxed{P_f(0,T) = \frac{F(0,T)}{X(0)} P_d(0,T)}$$

So a market FX forward quote plus domestic DF implies a foreign DF.

**Curve construction point:** This is an equation constraint linking curve objects across currencies. The book stresses FX forwards (and basis swaps) impose arbitrage constraints in multi-currency curve construction.

---

### 2.7 Why Multi-Curve Matters: Separation of Discount and Index Curves

The cross-currency curve construction discussion points out that traditional swap pricing implicitly treated LIBOR as the discount rate; but LIBOR contains bank credit risk and may not be a suitable proxy for "risk-free" discounting.

To add degrees of freedom needed to satisfy constraints like (6.38), define:
- A **discount curve** $P$ for discounting cashflows
- A **pseudo-discount / index curve** $P^{(L)}$ for generating forward LIBOR

The book explicitly states that a single yield curve is not always compatible with no-arbitrage constraints in cross-currency markets and that separating curves helps ensure linear instruments are correctly priced at time 0.

---

### 2.8 Cross-Currency (CRX) Yield Spread as a Curve-Level "Basis"

Measure the difference between index and discount curves via:

$$P^{(L)}(t) = P(t) e^{-s(t) t}$$

Solve for $s(t)$:

$$\boxed{s(t) = -\frac{1}{t} \ln\left(\frac{P^{(L)}(t)}{P(t)}\right)}$$

**Units:** $s(t)$ is a yield spread (units $1/\text{year}$, often quoted in bp).

**Interpretation:** Under Assumption 6.5.1, $s(t) = 0$ for USD, usually small for other currencies, but can blow out in stress.

---

### 2.9 Cross-Currency Basis Swap Par Condition as a Curve Constraint (Toy but Principled)

**Sources:**
- CRX basis swaps exchange floating LIBOR payments across currencies plus/minus a spread and exchange notionals at inception and maturity with ratio set to spot FX
- Standard swap valuation proceeds by projecting floating payments (often using forward rates) and discounting cashflows

We build a toy par condition for a CRX basis swap that is consistent with:
- Projecting foreign LIBOR using a foreign index curve $P_f^{(L)}$
- Discounting foreign cashflows using a foreign discount curve $P_f$

Let payment dates be $0 = t_0 < t_1 < \cdots < t_n = T$, accruals $\tau_i = t_{i+1} - t_i$. Define foreign projected forward LIBOR:

$$L_f^{\text{proj}}(0, t_i, t_{i+1}) = \frac{1}{\tau_i} \left( \frac{P_f^{(L)}(0,t_i)}{P_f^{(L)}(0,t_{i+1})} - 1 \right)$$

Consider a basis swap where the foreign leg pays $(L_f^{\text{proj}} + e)$ and the domestic leg is chosen as the "bedrock" with discount/index aligned (Assumption 6.5.1 style simplification).

Then a par condition (PV = 0) can be written (after scaling notionals by spot, per the notional-exchange convention) as:

$$\boxed{1 = \sum_{i=0}^{n-1} (L_f^{\text{proj}}(0,t_i,t_{i+1}) + e) \, \tau_i \, P_f(0,t_{i+1}) + P_f(0,t_n)} \tag{Par-basis-constraint}$$

**Interpretation:** Given $P_f^{(L)}$ (to compute $L_f^{\text{proj}}$) and basis quotes $e$, this equation constrains the discount curve $P_f$.

**Why this creates nonzero basis:** If $P_f^{(L)} \neq P_f$, then the PV of a floating leg is not automatically par; $e$ (and/or $P_f$) must adjust to satisfy par. This is the multi-curve mechanism behind basis.

---

## Measurement & Risk (Only What Belongs in Chapter 21)

### 3.1 CIP and the Spot–Forward–Curves Linkage

| Form | Formula |
|------|---------|
| Rate form (continuous) | $F_0 = S_0 e^{(r_d - r_f)T}$ |
| Discount-factor form | $F(0,T) = X(0) \, P_f(0,T) / P_d(0,T)$ |
| Inverse-quote form | $P_f(T) = P_d(T) / (X(0) Y(T))$ |

### 3.2 FX Forwards as Curve Constraints: What Do They Constrain?

Given spot $X(0)$ and two discount curves $P_d, P_f$, the no-arbitrage forward is pinned down. Conversely, given:
- A domestic discount curve $P_d$
- Spot $X(0)$
- Market forwards $F(0,T)$

you can infer the foreign discount factor $P_f(0,T)$. This is exactly the "cross-currency curve construction" constraint logic emphasized in the text.

### 3.3 Cross-Currency Basis: Concept and Quoting

The text distinguishes a curve-level basis via the yield spread $s(t)$ between discount and index curves:

$$P^{(L)}(t) = P(t) e^{-s(t) t}$$

with $s(t)$ typically small but capable of large moves in stress.

**Market quoting:** Cross-currency basis swaps are quoted as a spread $e$ added to one floating leg. The leg that receives/pays the spread and the sign convention are market-dependent (desk must confirm). The key structural fact is: these swaps exchange floating legs across currencies and exchange notionals at spot ratio.

### 3.4 Basis Swaps as Additional Constraints (When "Pure CIP" Is Not Enough)

- FX forwards are often liquid only out to ~1 year; to build curves out to long maturities one uses cross-currency basis swaps
- Failing to fit CRX basis swaps can create arbitrageable inconsistencies (the book sketches a multi-step conversion/hedging scheme linking USD cashflows into JPY cashflows via swaps and a CRX basis swap)

### 3.5 Curve Construction Implications (What Constrains What?)

1. **"Domestic curve + FX forwards $\Rightarrow$ foreign curve"**
   Using (6.38) or (CIP-DF), a forward quote gives a direct equation for $P_f(0,T)$ given $P_d(0,T)$.

2. But if you simultaneously want to fit local IRS markets (often quoting LIBOR-based swaps), you may need separate discount and index curves because swap pricing assumptions may not be consistent with FX forward constraints.

3. Cross-currency basis swaps then provide the extra market constraints (especially at longer maturities) to pin down the multi-curve system.

### 3.6 Risk Reporting Preview: FX Delta vs IR PV01 vs Basis Exposure

We preview the three main linear risk buckets for cross-currency instruments (all expressed in the reporting currency):

**FX delta:** Sensitivity to spot FX $X(0)$.
For an FX forward PV $V_0 = X(0) P_f - K P_d$:

$$\frac{\partial V_0}{\partial X(0)} = P_f(0,T) \quad \text{(units: domestic per unit change in } d/f\text{)}$$

**Interest-rate PV01 / DV01 by currency:** Sensitivity to shifts in discount curves.
Tuckman defines DV01 as the price change for a 1bp yield change:

$$\text{DV01} = -\frac{\partial P}{\partial y} \times 0.0001$$

For cross-currency portfolios, you compute analogous PV01s with respect to:
- Domestic curve nodes (domestic PV01)
- Foreign curve nodes (foreign PV01 converted into domestic using spot)
- Possibly key-rate PV01s (bucketed exposures)

**Cross-currency basis exposure:** Sensitivity to basis parameters (e.g., the spread $e$ in basis swaps or a curve-level $s(t)$).
This is separate from pure FX delta and pure IR PV01, and is exactly why basis markets exist: they represent a traded dimension not captured by single-curve CIP.

### 3.7 Collateral Currency Choice and Discounting (Preview Only)

The text notes uncollateralized derivatives require CVA etc. (out of scope) and discusses how, if you assume an index–discounting basis in one currency (e.g., USD), you can translate it into other currencies using FX forwards and cross-currency basis swap quotes. It also points out potential conflicts between discount-curve information sources (OIS vs CRX basis swap markets) can be resolved by analyzing collateral mechanisms, but that is outside scope.

---

## Worked Examples

*Rounding policy: values rounded to 6 decimals (rates) or 6–7 significant digits (DFs/FX) unless otherwise stated.*

---

### Example A — CIP Forward from Two Zero Rates (Continuous Compounding)

**Goal:** Given $S(0)$, domestic and foreign zero rates for maturity $T$, compute CIP-implied forward $F(0,T)$.

**Conventions (this example):**
- Domestic $d =$ USD, foreign $f =$ EUR
- Spot quote $S(0) = X(0) = 1.1000$ USD/EUR (USD per 1 EUR)
- Rates: continuous compounding (Hull's forward FX formula)
- $r_d = 3.00\% = 0.0300$, $r_f = 1.00\% = 0.0100$
- Maturity $T = 0.5$ years

**Step 1: Apply CIP (rate form)**

$$F(0,T) = S(0) \, e^{(r_d - r_f)T}$$

Compute exponent:
$$(r_d - r_f)T = (0.0300 - 0.0100) \cdot 0.5 = 0.0100$$

Compute $e^{0.0100} \approx 1.010050$.

**Step 2: Forward**
$$F(0, 0.5) = 1.1000 \times 1.010050 \approx 1.111055 \text{ USD/EUR}$$

**Forward points (interpretation):**
$$\text{Forward points} = F - S = 1.111055 - 1.1000 = 0.011055 \text{ USD/EUR}$$

**Unit check:** $F$ has units USD/EUR (same as spot). ✓

---

### Example B — CIP Using Discount Factors (Reconcile with Example A)

**Goal:** Use $F(0,T) = S(0) \, P_f(0,T) / P_d(0,T)$ and reconcile with Example A.

**Conventions:** Same as Example A; continuous-compounded DFs.

**Compute discount factors:**
$$P_d(0,0.5) = e^{-r_d T} = e^{-0.0300 \cdot 0.5} = e^{-0.0150} \approx 0.985112$$
$$P_f(0,0.5) = e^{-r_f T} = e^{-0.0100 \cdot 0.5} = e^{-0.0050} \approx 0.995012$$

**Apply DF-form CIP (forward FX rate):**
$$F(0,0.5) = S(0) \frac{P_f(0,0.5)}{P_d(0,0.5)} = 1.1000 \times \frac{0.995012}{0.985112}$$

**Compute ratio:**
$$\frac{0.995012}{0.985112} \approx 1.010050$$

**So:**
$$F(0,0.5) \approx 1.1000 \times 1.010050 = 1.111055$$

matching Example A (up to rounding). ✓

---

### Example C — Implied Foreign DF from FX Forward + Domestic DF

**Goal:** Given $S(0)$, $F(0,T)$, $P_d(0,T)$, infer $P_f(0,T)$ and an implied foreign zero rate.

**Conventions:** Same as A/B; $T = 0.5$.

**Given:**
- $S(0) = 1.1000$
- $F(0, 0.5) = 1.111055$
- $P_d(0, 0.5) = 0.985112$

**From $F = S \cdot P_f / P_d$:**
$$P_f(0, 0.5) = \frac{F(0, 0.5)}{S(0)} P_d(0, 0.5)$$

**Compute:**
$$\frac{F}{S} = \frac{1.111055}{1.1000} \approx 1.010050$$

**Then:**
$$P_f(0, 0.5) \approx 1.010050 \times 0.985112 = 0.995012$$

**Implied foreign continuous zero rate:**
$$r_f = -\frac{1}{T} \ln P_f(0,T) = -\frac{1}{0.5} \ln(0.995012)$$

$\ln(0.995012) \approx -0.0050$, so:
$$r_f \approx -2 \times (-0.0050) = 0.0100 = 1.00\%$$

---

### Example D — Multiple Maturities: Bootstrap an "FX-Implied" Foreign Curve

**Goal:** Using a strip of FX forwards plus a domestic discount curve, compute implied foreign DFs and an implied foreign zero curve (toy).

**Conventions:**
- Domestic USD, foreign EUR
- Spot $S(0) = 1.1000$ USD/EUR
- Domestic discount curve (continuous, flat $r_d = 3\%$):
  - $P_d(0, 0.5) = e^{-0.015} = 0.985112$
  - $P_d(0, 1) = e^{-0.03} = 0.970446$
  - $P_d(0, 2) = e^{-0.06} = 0.941765$
- Observed FX forwards (toy; assume market quotes $F(0,T)$ in USD/EUR):
  - $F(0, 0.5) = 1.111055$
  - $F(0, 1) = 1.122221$
  - $F(0, 2) = 1.144892$

**Step 1: Implied foreign discount factors**

Use:
$$P_f(0,T) = \frac{F(0,T)}{S(0)} P_d(0,T)$$

$T = 0.5$:
$$P_f(0, 0.5) = \frac{1.111055}{1.1000} \cdot 0.985112 = 1.010050 \cdot 0.985112 = 0.995012$$

$T = 1$:
$$P_f(0, 1) = \frac{1.122221}{1.1000} \cdot 0.970446 = 1.020201 \cdot 0.970446 \approx 0.990050$$

$T = 2$:
$$P_f(0, 2) = \frac{1.144892}{1.1000} \cdot 0.941765 = 1.0408108 \cdot 0.941765 \approx 0.980199$$

**Step 2: Implied foreign zero rates (continuous)**

$$r_f(T) = -\frac{1}{T} \ln P_f(0,T)$$

| Maturity | $P_f$ | $r_f$ |
|----------|-------|-------|
| 0.5Y | 0.9950 | ≈1.00% |
| 1Y | 0.9901 | ≈1.00% |
| 2Y | 0.9802 | ≈1.00% |

**Sanity checks:**
- DFs decreasing with maturity: $0.9950 > 0.9901 > 0.9802$ ✓
- Rates stable and consistent with the forward strip ✓

---

### Example E — Incorporating Cross-Currency Basis as an Additional Curve Constraint

**Goal:** Given a quoted basis spread $e$ on a foreign floating leg, show how it changes the implied relationship between a foreign projection curve and a foreign discount curve (toy bootstrap).

**Source-backed starting points:**
- Use a separate index curve $P^{(L)}$ to generate LIBOR forwards
- Basis swaps exchange floating legs plus a spread and exchange notionals at spot ratio; a one-period basis swap is an FX forward (conceptual link)
- Measuring curve difference via a yield spread $s$: $P^{(L)}(t) = P(t) e^{-s(t) t}$

**Conventions (toy):**
- Domestic USD is the "bedrock" (assume domestic index = domestic discount for simplicity; Assumption 6.5.1-style)
- Foreign JPY leg receives a basis spread $e$ added to foreign projected LIBOR. (Many markets quote the spread on one leg; desk must confirm sign/leg.)
- Payment dates: $t_1 = 0.5$, $t_2 = 1.0$; $\tau_0 = \tau_1 = 0.5$

**Inputs:**
- Foreign projection curve (JPY index curve):
  - $P_f^{(L)}(0, 0.5) = 0.9945$
  - $P_f^{(L)}(0, 1.0) = 0.9885$
- Foreign discount factor at 0.5y (assumed known from short-end instruments / FX forwards): $P_f(0, 0.5) = 0.9950$
- Quoted 1Y basis spread on the foreign leg: $e = -15$ bp $= -0.0015$

**Step 1: Compute projected forward LIBORs from $P_f^{(L)}$**

$$L_0 = \frac{1}{0.5} \left( \frac{1}{0.9945} - 1 \right) \approx 0.011064$$
$$L_1 = \frac{1}{0.5} \left( \frac{0.9945}{0.9885} - 1 \right) \approx 0.012142$$

**Step 2: Solve for the unknown foreign discount factor $P_f(0,1)$**

Using the par condition (Par-basis-constraint) for two periods:

$$1 = (L_0 + e) \cdot 0.5 \cdot P_f(0, 0.5) + (L_1 + e) \cdot 0.5 \cdot P_f(0, 1) + P_f(0, 1)$$

Rearrange:
$$1 - (L_0 + e) 0.5 P_f(0, 0.5) = [1 + (L_1 + e) 0.5] P_f(0, 1)$$

**Compute the left side:**
- $L_0 + e = 0.011064 - 0.0015 = 0.009564$
- $(L_0 + e) \cdot 0.5 \cdot P_f(0, 0.5) = 0.009564 \cdot 0.5 \cdot 0.9950 \approx 0.004758$
- Left side $= 1 - 0.004758 = 0.995242$

**Compute the bracket:**
- $L_1 + e = 0.012142 - 0.0015 = 0.010642$
- $1 + (L_1 + e) 0.5 = 1 + 0.010642 \cdot 0.5 = 1 + 0.005321 = 1.005321$

**So:**
$$P_f(0, 1) = \frac{0.995242}{1.005321} \approx 0.98998$$

**Step 3: Interpret as a curve-level basis $s(1)$**

Using $P^{(L)}(1) = P(1) e^{-s(1) \cdot 1}$ gives:

$$s(1) = -\ln\left(\frac{P^{(L)}(0, 1)}{P_f(0, 1)}\right) = -\ln\left(\frac{0.9885}{0.98998}\right) \approx 0.0015 = 15 \text{ bp}$$

This illustrates (toy) how a quoted basis spread leads to a nonzero $s$, i.e., a separation between projection and discount curves.

**Important:** The mapping between market-quoted basis swap spreads $e$ and curve-level spreads $s(t)$ is convention- and calibration-dependent. The above is a simplified illustration of the "constraint logic," not a universal market formula.

---

### Example F — Price an FX Forward Under Discounting; PV=0 at No-Arbitrage Forward

**Goal:** Value an FX forward as PV of two cashflows; show PV=0 at CIP forward and PV≠0 when traded forward differs.

**Conventions:**
- Domestic USD, foreign EUR
- Spot $X(0) = 1.1000$ USD/EUR
- Maturity $T = 1$
- Discount factors:
  - $P_d(0, 1) = 0.970446$
  - $P_f(0, 1) = 0.990050$
- Contract: receive 1 EUR at $T$, pay $K$ USD at $T$

**Step 1: Compute no-arbitrage forward $K^*$**

$$K^* = X(0) \frac{P_f}{P_d} = 1.1000 \times \frac{0.990050}{0.970446} = 1.1000 \times 1.020201 = 1.122221 \text{ USD/EUR}$$

**Step 2: PV formula**

$$V_0 = X(0) P_f - K P_d$$

**At $K = K^*$:**
- $X(0) P_f = 1.1000 \times 0.990050 = 1.089055$
- $K^* P_d = 1.122221 \times 0.970446 \approx 1.089055$

So $V_0 \approx 0$. ✓

**Step 3: Mispriced traded forward**

Suppose traded $K_{\text{mkt}} = 1.1250$. Then:
$$V_0 = 1.089055 - 1.1250 \times 0.970446$$

Compute $1.1250 \times 0.970446 = 1.091751$.

So:
$$V_0 \approx 1.089055 - 1.091751 = -0.002696 \text{ USD per EUR notional}$$

For 10,000,000 EUR notional: $V_0 \approx -26,960$ USD.

---

### Example G — Cross-Currency Basis Swap Par Condition (2 Periods) and Solve for Par Basis

**Goal:** Write PV equation for a toy 2-period CRX basis swap and solve the par basis spread $e$.

**Conventions (toy, but aligned to source structure):**
- Domestic currency $d =$ USD is reporting currency
- Foreign currency $f =$ JPY
- Spot FX $X(0) = 0.0091$ USD/JPY (USD per 1 JPY)
- Notional: $N_d = 1{,}000{,}000$ USD
- Foreign notional set by spot ratio (as per CRX basis swap description):
  $$N_f = N_d / X(0) = 1{,}000{,}000 / 0.0091 \approx 109{,}890{,}110 \text{ JPY}$$
- Payment dates: $t_1 = 0.5$, $t_2 = 1.0$; $\tau_0 = \tau_1 = 0.5$
- Domestic discount/index curve coincide (simplifying "bedrock" assumption): PV of domestic floating+principal leg ≈ par
- Foreign leg pays $(L_f^{\text{proj}} + e)$ where $L_f^{\text{proj}}$ comes from foreign index curve $P_f^{(L)}$
- Foreign discount curve $P_f$ is used to discount foreign cashflows

**Inputs:**
- Foreign discount DFs: $P_f(0, 0.5) = 0.9950$, $P_f(0, 1) = 0.9900$
- Foreign index (projection) DFs: $P_f^{(L)}(0, 0.5) = 0.9945$, $P_f^{(L)}(0, 1) = 0.9885$

**Step 1: Compute foreign forward LIBORs from $P_f^{(L)}$**

$$L_0 = \frac{1}{0.5} \left( \frac{1}{0.9945} - 1 \right) \approx 0.011064$$
$$L_1 = \frac{1}{0.5} \left( \frac{0.9945}{0.9885} - 1 \right) \approx 0.012142$$

**Step 2: Write PV in USD**

- Domestic USD floating leg PV $\approx N_d$ under the simplification that domestic index=discount. (This is the standard floating-leg telescoping identity when the same curve is used.)
- Foreign JPY leg PV (in JPY):

$$PV_f^{\text{JPY}} = N_f \left[ (L_0 + e) 0.5 P_f(0, 0.5) + (L_1 + e) 0.5 P_f(0, 1) + P_f(0, 1) \right]$$

Convert to USD using spot: $PV_f^{\text{USD}} = X(0) \, PV_f^{\text{JPY}}$.

Because $X(0) N_f = N_d$ by notional ratio, the USD PV becomes:

$$PV_f^{\text{USD}} = N_d \left[ (L_0 + e) 0.5 P_f(0, 0.5) + (L_1 + e) 0.5 P_f(0, 1) + P_f(0, 1) \right]$$

Thus swap PV (receive USD, pay JPY) is:
$$PV_{\text{swap}} = N_d - PV_f^{\text{USD}}$$

Par requires $PV_{\text{swap}} = 0$, so:
$$1 = (L_0 + e) 0.5 P_{0.5} + (L_1 + e) 0.5 P_1 + P_1$$

where $P_{0.5} = 0.9950$, $P_1 = 0.9900$.

**Step 3: Compute the "no-basis" PV term**

Interest PV without basis:
- Period 1: $L_0 \cdot 0.5 \cdot 0.9950 \approx 0.011064 \cdot 0.4975 \approx 0.005504$
- Period 2: $L_1 \cdot 0.5 \cdot 0.9900 \approx 0.012142 \cdot 0.4950 \approx 0.006010$

So:
$$B = 0.005504 + 0.006010 + 0.9900 = 1.001515$$

**Basis annuity:**
$$A = 0.5 \cdot 0.9950 + 0.5 \cdot 0.9900 = 0.9925$$

**Par condition:**
$$1 = B + eA \quad \Rightarrow \quad e = \frac{1 - B}{A} = \frac{1 - 1.001515}{0.9925} \approx -0.001526$$

**Final answer:**
$$\boxed{e_{\text{par}} \approx -0.001526 \approx -15.26 \text{ bp}}$$

**Sanity check:** $e < 0$ because $B > 1$: projected floating leg PV (discounted with $P_f$ but projected with $P_f^{(L)}$) is slightly above par, so the spread must reduce it to par. ✓

---

### Example H — Basis Shock Sensitivity (+5bp) on a Basis Swap

**Goal:** Apply a +5bp change in basis, compute PV change, interpret as basis risk.

Use Example G setup and par basis.

From Example G, swap PV per $N_d$ is:
$$PV = N_d [1 - (B + eA)]$$

So sensitivity to $e$:
$$\frac{\partial PV}{\partial e} = -N_d A$$

With $N_d = 1{,}000{,}000$, $A = 0.9925$, and $\Delta e = +5$ bp $= +0.0005$:

$$\Delta PV \approx -1{,}000{,}000 \times 0.9925 \times 0.0005 = -496.25 \text{ USD}$$

**Interpretation:** A +5bp widening of the spread on the foreign leg reduces PV by about $496 for a $1mm notional in this toy. That is basis risk distinct from FX spot and IR curve moves.

---

### Example I — Hedged Foreign Bond Valuation (Where Basis Can Enter)

**Goal:** Value a foreign-currency bond hedged back into domestic currency using FX forwards; show dependence on curves and forward quotes.

**Conventions:**
- Domestic USD, foreign JPY
- Spot $X(0) = 0.0091$ USD/JPY
- Foreign ZCB payoff: $N_f = 100{,}000{,}000$ JPY at $T = 1$
- Discount factors: $P_f(0, 1) = 0.9900$, $P_d(0, 1) = 0.970446$
- Hedge: sell JPY forward at $K$ USD/JPY for delivery at $T$

**Unhedged USD PV**

Foreign bond PV in USD:
$$PV_{\text{unhedged}} = X(0) \, N_f \, P_f(0, 1)$$

Compute:
- $X(0) N_f = 0.0091 \times 100{,}000{,}000 = 910{,}000$ USD
- Multiply by $P_f = 0.9900$:

$$PV_{\text{unhedged}} = 910{,}000 \times 0.9900 = 900{,}900 \text{ USD}$$

**Hedged USD PV using a forward**

At maturity, forward converts $N_f$ JPY into $K N_f$ USD. PV:
$$PV_{\text{hedged}} = (K N_f) \, P_d(0, 1)$$

**No-arbitrage forward (CIP)**
$$K^* = X(0) \frac{P_f(0, 1)}{P_d(0, 1)} = 0.0091 \times \frac{0.9900}{0.970446} \approx 0.0091 \times 1.020201 \approx 0.00928383 \text{ USD/JPY}$$

**Compute hedged PV at $K^*$:**
- $K^* N_f \approx 0.00928383 \times 100{,}000{,}000 = 928{,}383$ USD at $T$
- Discount: $928{,}383 \times 0.970446 \approx 900{,}900$ USD

So $PV_{\text{hedged}} = PV_{\text{unhedged}}$ under CIP. ✓

**Where basis enters:**

If the traded forward differs from $K^*$ (e.g., due to cross-currency basis / curve segmentation), then hedged PV changes.

Example: traded $K_{\text{mkt}} = 0.0093000$. Then:
$$PV_{\text{hedged,mkt}} = 0.0093000 \times 100{,}000{,}000 \times 0.970446 = 930{,}000 \times 0.970446 \approx 902{,}514 \text{ USD}$$

Difference vs CIP PV:
$$902{,}514 - 900{,}900 = 1{,}614 \text{ USD}$$

This "extra" PV is a direct manifestation of forward/basis deviation from CIP-implied pricing.

---

### Example J — Arbitrage-Consistency Sanity Check (Forward-Implied Curve vs Independent Curve)

**Goal:** Domestic DF + FX forward strip implies foreign DF strip; compare implied foreign DF against an independently built foreign OIS curve; quantify mismatch and interpret.

**Conventions:**
- Domestic USD, foreign EUR
- Spot $S(0) = 1.1000$ USD/EUR
- Domestic discount curve: $P_d(0, 2) = 0.941765$ (as in Example D)
- Observed 2Y forward: suppose market quotes $F_{\text{mkt}}(0, 2) = 1.1500$ USD/EUR (toy)

**FX-implied foreign DF:**
$$P_f^{\text{FX}}(0, 2) = \frac{F_{\text{mkt}}(0, 2)}{S(0)} P_d(0, 2)$$

**Independently built foreign OIS curve DF (toy):** $P_f^{\text{OIS}}(0, 2) = 0.980199$ (≈1% continuous rate, as in Example D)

**Step 1: Compute FX-implied DF**
$$\frac{F}{S} = \frac{1.1500}{1.1000} = 1.045454545$$

So:
$$P_f^{\text{FX}}(0, 2) = 1.045454545 \times 0.941765 \approx 0.984572$$

**Step 2: Compare to independent foreign OIS DF**
$$\Delta P = P_f^{\text{FX}} - P_f^{\text{OIS}} = 0.984572 - 0.980199 = 0.004373$$

**Convert to implied 2Y continuous zero rates:**

FX-implied:
$$r_f^{\text{FX}}(2) = -\frac{1}{2} \ln(0.984572) \approx 0.00775 = 0.775\%$$

OIS:
$$r_f^{\text{OIS}}(2) = -\frac{1}{2} \ln(0.980199) = 1.00\%$$

**Mismatch in yield terms:** About $-22.5$ bp (FX-implied lower yield / higher DF).

**Interpretation (carefully labeled):**
- **Source-backed:** The book warns that building curves independently from local swaps will generally violate cross-currency no-arbitrage constraints such as (6.38).
- **Reasoned inference:** The mismatch between $P_f^{\text{FX}}$ and $P_f^{\text{OIS}}$ can be interpreted as a "basis" or segmentation/collateral effect that must be reconciled by a consistent multi-curve calibration (often involving basis swaps at longer maturities).

---

## Practical Notes

### Quoting Conventions and Common Ambiguity Traps

| Issue | Description |
|-------|-------------|
| **FX quote direction** | e.g., USD per EUR vs EUR per USD — flips formulas. If your market quote is the inverse direction, you must invert spot/forward before applying the formulas. Hull explicitly notes many spot rates are quoted as "units of currency per USD," which changes how you apply parity formulas. |
| **Spot date vs forward maturity** | I'm not sure about the exact spot/settlement conventions because they are currency-pair specific (spot lag, holidays, business day rules, fixing conventions). To be certain you must specify the currency pair and market standard (e.g., T+2 vs T+1, deliverable vs NDF, fixing source). |
| **Day count and compounding mismatches** | LIBOR-style rates are simple and depend on day count (often Actual/360); zero rates may be handled with continuous compounding in analytic formulas. The book also notes an important implementation nuance: the year fractions defining payment accruals can differ slightly from those defining forward LIBOR rates due to date adjustment conventions. |
| **Collateral currency** | The text highlights that discounting choices (OIS vs other) and collateral mechanisms can matter and can resolve conflicts between different sources of discount-curve information, but treats the full discussion as out of scope. |
| **Basis swap quoting** | I'm not sure: this is market-convention specific (e.g., whether the spread is added to the USD leg or the non-USD leg, and the sign convention). To be certain, specify the currency pair and quoting convention used by your broker/venue. |

### Implementation Pitfalls

| Pitfall | Notes |
|---------|-------|
| **Inconsistent calendars** | I'm not sure (book does not give full operational calendars here). In practice, inconsistent calendars lead to accrual mismatches and hedging noise; you need currency-specific holiday calendars and rolling conventions. |
| **Interpolation artifacts** | Curve construction requires interpolation; the text discusses interpolation artifacts and how perturbations can introduce noise, motivating careful technique choice. |
| **Curve dependency graph** | In multi-currency, "domestic curve + FX forwards ⇒ foreign curve" can create circularities if you also use cross-currency basis swaps to infer discounting vs index curve bases across currencies. The book warns about circularity and recommends anchoring the basis in one currency using domestic instruments. |

### Verification Tests

| Test | What to Check |
|------|---------------|
| **Dimension/unit checks** | $X$ in $d/f$; $P$ dimensionless; $L$ in 1/year; PV in $d$ |
| **PV(Forward)=0 at no-arb forward** | Check $V_0 = X(0) P_f - K P_d$ equals 0 at $K = F(0,T)$ |
| **Monotonicity/positivity** | $0 < P(0,T) \leq 1$; decreasing in $T$ under positive rates |
| **Small-bump stability** | Bump FX forwards slightly; implied $P_f$ should change smoothly (watch for interpolation-induced kinks) |

---

## Summary & Recall

### 10-Bullet Executive Summary

1. **CIP links spot FX, forward FX, and interest rates/discount factors** across two currencies.

2. **In discount-factor form**, the forward FX rate satisfies $F(0,T) = X(0) \, P_f(0,T) / P_d(0,T)$.

3. **In rate form** (continuous compounding), $F_0 = S_0 e^{(r_d - r_f)T}$.

4. **FX forwards act as curve constraints**: given $X(0)$ and $P_d$, a forward quote implies $P_f$, and vice versa.

5. **Multi-currency curve building must respect arbitrage constraints** from FX forwards and cross-currency basis swaps.

6. **Building domestic and foreign curves independently** from swaps can violate the FX-forward constraint (e.g., (6.38)), leading to cross-currency arbitrage.

7. **To reconcile constraints**, the book motivates separating discount curves from index (LIBOR) curves.

8. **Cross-currency "basis"** can be expressed at the curve level via $P^{(L)}(t) = P(t) e^{-s(t) t}$, defining a CRX yield spread $s(t)$.

9. **CRX basis swaps** exchange floating legs across currencies plus/minus a spread and exchange notionals at spot ratio; a one-period basis swap is an FX forward.

10. **Risk decomposes** into FX delta, IR PV01 by currency, and basis exposure; DV01/PV01 definitions generalize naturally to multi-currency reporting.

---

### Cheat Sheet of Formulas

| Formula | Expression |
|---------|------------|
| **CIP (rate form, continuous)** | $F_0 = S_0 e^{(r_d - r_f)T}$ (quote direction matters) |
| **CIP (discount-factor form)** | $F(0,T) = X(0) \dfrac{P_f(0,T)}{P_d(0,T)}$ |
| **FX forward PV** | $V_0 = X(0) P_f(0,T) - K P_d(0,T)$ |
| **Implied foreign DF** | $P_f(0,T) = \dfrac{F(0,T)}{X(0)} P_d(0,T)$ |
| **Forward LIBOR from index curve** | $L(0, T_i, T_{i+1}) = \dfrac{1}{\tau_i} \left( \dfrac{P^{(L)}(0,T_i)}{P^{(L)}(0,T_{i+1})} - 1 \right)$ |
| **Cross-currency yield spread** | $P^{(L)}(t) = P(t) e^{-s(t) t}$, so $s(t) = -\dfrac{1}{t} \ln\left(\dfrac{P^{(L)}(t)}{P(t)}\right)$ |
| **Basis swap par constraint (toy)** | $1 = \sum_i (L_f^{\text{proj}} + e) \tau_i P_f(0, t_{i+1}) + P_f(0,T)$ |

---

### Flashcards (30 Q/A)

| # | Question | Answer |
|---|----------|--------|
| 1 | What does CIP relate? | Spot FX, forward FX, and interest-rate discounting across two currencies. |
| 2 | CIP in DF form (domestic per foreign quote)? | $F(0,T) = X(0) \, P_f(0,T) / P_d(0,T)$ |
| 3 | CIP in Hull's rate form (continuous compounding)? | $F_0 = S_0 e^{(r_d - r_f)T}$ |
| 4 | What is $P_d(0,T)$? | Domestic discount factor: value at 0 of 1 unit domestic paid at $T$. |
| 5 | What is $X(0)$ in this chapter? | Domestic currency per 1 unit foreign ($d/f$). |
| 6 | What is the domestic price of 1 unit of foreign ZCB paying 1 foreign at $T$? | $X(t) P_f(t,T)$ |
| 7 | How do FX forwards constrain curves? | Given $X(0)$ and $P_d$, forward quotes imply $P_f$ via CIP. |
| 8 | What is equation (6.38) expressing? | No-arbitrage constraint linking USD DF, JPY DF, spot FX, and an FX forward quote. |
| 9 | What is an "FX-implied curve"? | Foreign discount factors derived from domestic discount factors and FX forward quotes. |
| 10 | Why can building curves independently create arbitrage? | Independently built curves may violate FX-forward parity constraint (6.38). |
| 11 | Why separate discount and forward/index curves? | To satisfy cross-currency no-arbitrage constraints and because LIBOR may not be risk-free for discounting. |
| 12 | Define pseudo-discount/index curve $P^{(L)}$. | Curve used so that forward LIBORs satisfy $L = \frac{1}{\tau}\left(\frac{P^{(L)}(T)}{P^{(L)}(T+\tau)} - 1\right)$ |
| 13 | Define CRX yield spread $s(t)$. | $P^{(L)}(t) = P(t) e^{-s(t) t}$ |
| 14 | Typical magnitude of $s(t)$ per text? | Usually a few basis points or less, but can blow out. |
| 15 | What is a cross-currency basis swap? | Floating–floating swap exchanging LIBOR legs in two currencies plus/minus a spread, with notional exchanges at inception/maturity. |
| 16 | What is special about a one-period CRX basis swap? | It is identical to an FX forward contract. |
| 17 | Why are basis swaps important for long maturities? | FX forward market is rarely liquid beyond ~1 year. |
| 18 | FX forward PV decomposition? | PV = PV(foreign receipt) − PV(domestic payment) = $X(0) P_f - K P_d$ |
| 19 | What does "basis" represent economically (high-level)? | A traded deviation from single-curve CIP due to curve segmentation/credit/liquidity/collateral effects. |
| 20 | What risk buckets matter for cross-currency instruments? | FX delta, IR PV01 by currency, basis exposure. |
| 21 | What is DV01 per Tuckman? | Price change for a 1bp yield change: $\text{DV01} = -\partial P/\partial y \times 0.0001$ |
| 22 | What is key-rate exposure conceptually? | PV sensitivity to changes at specific maturities/forward-rate buckets. |
| 23 | Why is quote direction crucial in FX formulas? | Inverting the quote inverts the parity relationship (forward points sign flips). |
| 24 | What can go wrong with accrual fractions? | Payment accrual fractions can differ from those defining forward LIBOR due to date adjustments. |
| 25 | What does $P^{(L)} \neq P$ imply? | Forward projection differs from discounting; multi-curve valuation needed. |
| 26 | How do you infer $P_f(0,T)$ from $F(0,T)$ and $P_d(0,T)$? | $P_f = (F/X) P_d$ (when quotes are consistent). |
| 27 | What is the "bedrock USD" assumption in the text? | Assumption 6.5.1: USD index curve equals USD discount curve (pre-crisis convention). |
| 28 | Why can circularity arise in discounting basis estimation? | If you use cross-currency instruments to infer USD basis, you can create a circular dependency; text suggests using domestic instruments to anchor. |
| 29 | What does failing to fit basis swaps risk? | Arbitrageable inconsistencies across currencies. |
| 30 | What is the core curve-construction "constraint logic" takeaway? | FX forwards and CRX basis swaps supply market equations that link discount/index curves across currencies; curves must be calibrated jointly. |

---

## Mini Problem Set (16 Questions)

*Increasing difficulty. Solution sketches provided for questions 1–8 only.*

---

**1.** Given $S(0) = 1.25$ USD/EUR, $r_d = 4\%$, $r_f = 1.5\%$, $T = 1$ (continuous), compute $F(0,1)$.

**Sketch:** Use $F = S e^{(r_d - r_f)T} = 1.25 \times e^{0.025} \approx 1.2817$.

---

**2.** Given $X(0) = 1.10$, $P_d(0, 2) = 0.94$, $P_f(0, 2) = 0.98$, compute $F(0, 2)$.

**Sketch:** Use $F = X \cdot P_f / P_d = 1.10 \times 0.98 / 0.94 \approx 1.1468$.

---

**3.** Given $X(0)$, $F(0,T)$, and $P_d(0,T)$, infer $P_f(0,T)$.

**Sketch:** Rearrange CIP: $P_f = (F/X) P_d$. (Algebra from CIP-DF.)

---

**4.** An FX forward receives 1 foreign at $T$ and pays $K$ domestic. Write PV at time 0 and solve for $K$ that makes PV=0.

**Sketch:** PV $= X P_f - K P_d$; set to zero $\Rightarrow K = X P_f / P_d$.

---

**5.** Quote inversion: If the market quotes EUR/USD instead of USD/EUR, how must the CIP formula be adapted?

**Sketch:** Invert the FX quote first; Hull notes FX quote conventions vary and can be "per USD," requiring inversion.

---

**6.** Using $P^{(L)}(t) = P(t) e^{-s(t) t}$, solve for $s(t)$ given $P^{(L)}(t)$ and $P(t)$.

**Sketch:** $s(t) = -(1/t) \ln(P^{(L)}/P)$.

---

**7.** Compute a forward LIBOR $L(0, T, T+\tau)$ given $P^{(L)}(0,T)$ and $P^{(L)}(0, T+\tau)$.

**Sketch:** Use $L = \frac{1}{\tau}\left(\frac{P^{(L)}(T)}{P^{(L)}(T+\tau)} - 1\right)$.

---

**8.** Explain qualitatively why independent curve construction in each currency can create arbitrage.

**Sketch:** The text shows (6.38) is required; independently estimated curves likely violate it, creating cross-currency arbitrage.

---

**9.** Build a 3-point FX-implied foreign curve from a domestic curve and a forward strip; check monotonicity of implied foreign DFs.

---

**10.** In a two-curve setup, show that if projection and discount curves coincide, the PV of a floating leg with principal repayment telescopes to par.

---

**11.** Given a basis swap quote $e$ on the foreign leg and a foreign index curve $P_f^{(L)}$, set up the calibration equation for foreign discount factors $P_f$ at a single maturity.

---

**12.** For an FX forward, compute FX delta and domestic PV01 (bump domestic curve by 1bp) for a given notional.

---

**13.** Construct a toy scenario where forward-implied foreign DFs conflict with an independently built foreign OIS curve; interpret the discrepancy.

---

**14.** Discuss how lack of FX forward liquidity beyond 1 year motivates using basis swaps for long-dated constraints.

---

**15.** Identify which market instruments anchor the USD discount vs index curve basis per the text's discussion (OIS, Fed funds/LIBOR basis swaps).

---

**16.** Describe a verification checklist for a multi-currency curve build (PV checks, monotonicity, quote-direction checks, bump stability).

---

## Source Map

### (A) Verified Facts — Cite Specific Sources

| Fact | Source |
|------|--------|
| CIP rate form: $F_0 = S_0 e^{(r_d - r_f)T}$ | Hull Ch 5 |
| CIP discount-factor form: $F(0,T) = X(0) P_f / P_d$ | Andersen Vol 1 |
| FX forward replication via foreign/domestic ZCBs | Andersen Vol 1 |
| Multi-curve separation: discount vs index curves | Andersen Vol 1 Ch 6 |
| Cross-currency basis swaps exchange floating + spread with notional exchanges | Andersen Vol 1 |
| One-period CRX basis swap = FX forward | Andersen Vol 1 |
| Independent curve construction can violate (6.38) | Andersen Vol 1 Ch 6 |
| CRX yield spread definition: $P^{(L)}(t) = P(t) e^{-s(t) t}$ | Andersen Vol 1 Ch 6 |
| DV01 definition | Tuckman Ch 5 |
| FX quote direction warning | Hull Ch 5 |

### (B) Reasoned Inference — Note Derivation Logic

| Inference | Derivation |
|-----------|------------|
| Implied foreign DF formula: $P_f = (F/X) P_d$ | Algebra from CIP-DF |
| Yield spread formula: $s(t) = -(1/t) \ln(P^{(L)}/P)$ | Algebra from yield spread definition |
| Par basis swap constraint equation | Constructed from swap valuation principles + multi-curve setup |
| Basis sensitivity formula | Differentiation of swap PV w.r.t. spread |

### (C) Speculation — Flag Uncertainties

| Topic | Uncertainty |
|-------|-------------|
| Spot/settlement conventions | Currency-pair specific; not fully specified in sources |
| Basis swap spread conventions (which leg, sign) | Market-convention specific; varies by venue |
| Cross-currency calendar construction | Book does not provide full operational calendars |
| Collateral currency discounting details | Flagged as "out of scope" in sources |

---

*Last Updated: January 2026*
