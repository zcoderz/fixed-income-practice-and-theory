# Chapter 31: Multi-Currency Risk — FX Delta + Rates Delta (by currency) + Basis Risk

---

## Fact Classification

### (A) Verified Facts (Source-Backed)

- FX forward under deterministic rates: $F_0 = S_0 e^{(r_d - r_f)T}$ under the $S_0 = d/f$ convention (Hull)
- FX forward in discount-factor form: $F_0 = S_0 \frac{P_f(0,T)}{P_d(0,T)}$ (derived from Hull's parity)
- Delta as partial derivative of value: $\Delta = \partial V / \partial X$ (Hull)
- Multi-curve framing: discounting and forecasting curves can be separated (Andersen & Piterbarg)
- Sensitivity measurement through perturbations to market instruments: funding instruments → discount sensitivity; index instruments → forecasting sensitivity; basis instruments → basis sensitivity (Andersen & Piterbarg)
- Cross-currency basis swap PV form and par basis definition (Andersen & Piterbarg)
- Cross-currency yield spread definition: $P^{(L)}(t) = P(t) e^{-s(t)t}$ (Andersen & Piterbarg)
- DV01 finite-difference definition: $\text{DV01} = P(y - 1\text{bp}) - P(y)$ (Tuckman)
- Discount-factor curve as $P(t,T)$ "zero-bond curve" (Tuckman)
- OIS discounting usage and post-crisis OIS shift (Hull)
- Hull explicitly warns about FX quote direction differences across pairs

### (B) Reasoned Inference (Derived from A)

- Generic domestic-currency PV decomposition: $PV_d = \sum_k C_{d,k} P_d(0,T_k) + S_0 \sum_j C_{f,j} P_f(0,T_j)$ (natural generalization of discount-factor valuation + FX conversion logic)
- FX01 conversion: $FX01 = 0.01 \cdot S_0 \cdot \Delta_{FX}$
- Basis DV01 formula as the derivative of the basis-swap PV expression with respect to $e$
- Hedge mapping "what to hedge with what" as practitioner interpretation of risk buckets

### (C) Speculation (Clearly Labeled; Minimal)

- I'm not sure about cross-currency basis quoting/sign conventions across different markets and systems (which leg carries "+$e$" and sign in risk reports) without the currency pair, trade direction, and desk conventions
- I'm not sure about collateral currency/CSA-driven discounting choice and how it is operationalized without CSA and clearing details
- I'm not sure about exact FX swap settlement/calendar conventions for specific pairs without pair/market specification

---

## Conventions & Notation

### Notation Glossary

| Symbol | Meaning | First Defined |
|--------|---------|---------------|
| $d$, $f$ | Domestic and foreign currency labels | This chapter |
| $S_0$ or $S_t$ | Spot FX at time $t$, quoted $d$ per 1 $f$ (e.g., USD per 1 EUR) | This chapter |
| $F_{0,T}$ or $F_{t,T}$ | FX forward for settlement at $T$, quoted $d$ per 1 $f$ | This chapter |
| $P_d(0,T)$, $P_f(0,T)$ | Discount factors in currencies $d$, $f$ | Ch 2, extended |
| $P_d^{(L)}(0,T)$, $P_f^{(L)}(0,T)$ | Index/projection "pseudo-discount" curves for $L$-indexed legs | This chapter |
| $L_d(0, t_i, t_{i+1})$, $L_f(0, t_i, t_{i+1})$ | Forward floating rates for accrual period $[t_i, t_{i+1}]$ | This chapter |
| $e$ | Cross-currency basis spread (quoted; convention-dependent) | This chapter |
| $\Delta_{FX}$ | FX delta: $\partial PV_d / \partial S_0$ (units: $f$) | This chapter |
| $PV01_{d,\text{disc}}$, $PV01_{f,\text{disc}}$ | PV01s to domestic/foreign discount curves | This chapter |
| $PV01_{\text{basis}}$ | Basis DV01 to the quoted cross-currency basis spread | This chapter |
| $DV01$ | Bond DV01 as in Tuckman | Ch 11 |
| $\tau$ or $\alpha$ | Year fraction / accrual factor | Ch 1, 4 |

### Defaults Used in Examples

- **Reporting currency:** Domestic $d$ = USD
- **FX quote direction:** $S = d/f$ = "domestic per 1 foreign" (e.g., USD per EUR)
- **Curves:** Treated as inputs (no bootstrapping in this chapter)
- **Risk decomposition:** First-order buckets — FX delta, rate PV01 (by curve/currency), and basis DV01
- **Bump definition:** $P'(0,T) = P(0,T) \exp(-0.0001 \cdot T)$ (equivalent to +1 bp parallel shift of continuously-compounded zero rates)

### Key Convention Warnings

> **FX Quote Direction:** Hull notes that actual market quote conventions can differ by pair; if the market quotes the inverse, you must invert and apply the chain rule for deltas.

> **Cross-Currency Basis Convention:** Which leg receives/pays the spread and the sign of the quoted "basis" differs across markets/venues. If your desk uses the opposite quoting convention, sensitivities will flip sign. I'm not sure which convention you want without knowing the pair and desk quoting; you must specify (i) "basis added to non-USD leg" vs "basis added to USD leg", and (ii) pay/receive sign in your risk system.

---

## Core Concepts

### 1) PV in a Reporting Currency

**Formal Definition:**

A multi-currency portfolio is reported in a single currency $d$. Each cashflow in currency $c \in \{d, f\}$ is discounted on the curve appropriate for that currency and then converted to $d$ (if needed) using the spot FX quote convention.

**Intuition:**

A PV is just a sum of present values of deterministic (or expected) cashflows. Multi-currency adds exactly one extra step: currency conversion.

**Trading / Risk / Portfolio Practice:**

Your P&L explain is cleaner if your system:
1. Values each currency leg in its native currency using its curve(s), then
2. Converts to the reporting currency at spot (or at an agreed conversion framework), and
3. Reports FX delta and rate PV01s separately

---

### 2) FX Forward Parity / Arbitrage Consistency

**Formal Definition:**

Under Hull's convention $S_0 = d/f$, the no-arbitrage forward is:

$$\boxed{F_0 = S_0 e^{(r_d - r_f)T}}$$

with continuous compounding, and he explicitly warns about quote direction differences across pairs.

**Intuition:**

If $d$ rates exceed $f$ rates, the $f$ currency tends to trade at a forward premium in $d/f$ terms, because holding $d$ earns more interest.

**Trading / Risk / Portfolio Practice:**

For risk, an FX forward is not "pure FX": it also embeds the interest differential (and in modern markets, potentially cross-currency basis). This is why FX hedges can introduce (or shift) rate PV01s.

---

### 3) Discount Curve vs Projection (Forward-Rate) Curve

**Formal Definition:**

The interest-rate-modeling text emphasizes that "the term structure" is not a single curve in modern markets: discounting and forecasting can be separated, yielding (at least) a discount curve and one or more forward (projection) curves.

**Intuition:**

You discount cashflows using a curve consistent with funding/collateralization; you project floating coupons using a curve consistent with the floating index (e.g., LIBOR tenor).

**Trading / Risk / Portfolio Practice:**

This matters for risk attribution:
- "Discount PV01" and "projection PV01" can point in different directions
- The same hedge trade can reduce one but increase the other

---

### 4) FX Delta

**Formal Definition:**

Delta is a partial derivative of value with respect to an underlying price: $\Delta = \partial V / \partial X$.

In multi-currency reporting, FX delta is:

$$\boxed{\Delta_{FX} := \frac{\partial PV_d}{\partial S_0}}$$

where $S_0$ is the spot FX in your chosen quote direction.

**Intuition:**

"How much does my PV change if FX moves a tiny bit, with curves held fixed?"

**Trading / Risk / Portfolio Practice:**

FX delta is typically hedged with:
- Spot/forwards, or
- FX swaps (for funding/rolling hedges)

depending on settlement horizon and how you manage cash.

---

### 5) Rate PV01 by Currency (and by Curve)

**Formal Definition:**

The fixed-income literature defines DV01/PV01 as a price change under a 1 bp yield move; Tuckman gives a concrete finite-difference definition of DV01.

In multi-curve practice, we refine this into:
- $PV01_{d,\text{disc}}$: PV change under a +1 bp bump to the domestic discount curve
- $PV01_{d,\text{proj}}$: PV change under a +1 bp bump to the domestic projection curve (holding discount fixed)
- Similarly for foreign currency $f$

The interest-rate-modeling text frames curve risk through perturbations to the market instruments used to build curves: funding instruments drive discount sensitivity; index instruments drive forecasting sensitivity; basis instruments drive basis sensitivity.

**Intuition:**

Rate PV01 answers: "If the curve for currency $c$ shifts up 1 bp (in whatever bump definition you use), what happens to my PV in reporting currency?"

**Trading / Risk / Portfolio Practice:**

Hedged with swaps/futures/bonds in the relevant currency curve bucket.

---

### 6) Cross-Currency Basis and Basis Risk

**Formal Definition:**

A cross-currency basis swap exchanges floating payments in two currencies, with a quoted spread (basis) added to one leg. The interest-rate-modeling text gives the valuation of a USD/JPY basis swap where USD receives Libor flat and pays JPY Libor plus spread $e_{¥}$, and it defines "par" basis quotes as those making the swap PV zero.

**Intuition:**

Cross-currency basis is the market's way of reconciling:
- Two rate curves (domestic and foreign), and
- The FX forward market

in a world where simple covered-interest-parity relationships can be "broken" by funding/credit/collateral segmentation (the text motivates this through arbitrage-consistency arguments).

**Trading / Risk / Portfolio Practice:**

Basis DV01 measures sensitivity to the quoted basis spread. It is hedged with:
- Cross-currency basis swaps (same convention/tenor), or
- A replication portfolio of swaps + FX forwards that reproduces basis exposure

---

### 7) Cross-Currency Yield Spread (Index vs Discount in a Currency)

**Formal Definition:**

In the multi-currency formalism, the text defines a "cross-currency yield spread" $s(t)$ between a pseudo-discount curve $P^{(L)}(t)$ and a discount curve $P(t)$ via:

$$P^{(L)}(t) = P(t) \, e^{-s(t)t}$$

**Intuition:**

This is a convenient way to quantify "how far apart" the discounting and projection curves are, in yield terms.

**Trading / Risk / Portfolio Practice:**

If your system uses separate curves, your PV01 can be highly sensitive to which curve you bump and how you rebuild.

---

## Math and Derivations

### 2.1 PV of Multi-Currency Deterministic Cashflows in Domestic Currency

Assume deterministic cashflows $\{(C_{d,k}, T_{d,k})\}$ in currency $d$ and $\{(C_{f,j}, T_{f,j})\}$ in currency $f$. Let $P_d(0,T)$, $P_f(0,T)$ be discount factors in their own currencies.

A natural decomposition is:

$$\boxed{PV_d = \sum_k C_{d,k} \, P_d(0, T_{d,k}) \;+\; S_0 \sum_j C_{f,j} \, P_f(0, T_{f,j})}$$

**Unit Check:**
- $C_{d,k}$ is in $d$; $P_d$ is dimensionless $\Rightarrow$ term is in $d$
- $C_{f,j}$ is in $f$; $P_f$ dimensionless; $S_0$ is $d/f$ $\Rightarrow$ $S_0 C_{f,j} P_f$ is in $d$ ✓

**Connection to Arbitrage-Consistency:**

The interest-rate-modeling text relates FX forwards and discount factors in a USD/JPY setting, showing how inconsistency would imply arbitrage.

---

### 2.2 FX Forward Pricing (Discount-Factor Form)

Hull gives:

$$F_0 = S_0 e^{(r_d - r_f)T}$$

under $S_0 = d/f$ and continuous compounding.

Using $P_d(0,T) = e^{-r_d T}$, $P_f(0,T) = e^{-r_f T}$, we get:

$$F_0 = S_0 \frac{e^{-r_f T}}{e^{-r_d T}} = S_0 \frac{P_f(0,T)}{P_d(0,T)}$$

This matches the general structure that forward FX depends on the ratio of foreign to domestic discounting.

---

### 2.3 FX Delta for Deterministic Multi-Currency Cashflows

Start from:

$$PV_d(S_0) = \sum_k C_{d,k} P_d(0, T_{d,k}) + S_0 \sum_j C_{f,j} P_f(0, T_{f,j})$$

Differentiate w.r.t. $S_0$:

$$\boxed{\Delta_{FX} = \frac{\partial PV_d}{\partial S_0} = \sum_j C_{f,j} P_f(0, T_{f,j})}$$

**Units:**
- RHS is in $f$
- LHS: $PV_d$ is in $d$, $S_0$ is $d/f$, so $\partial PV_d / \partial S_0$ is $f$ ✓

**"Per 1% Move" Version (Desk-Friendly):**

For a small relative move $\Delta S / S$:

$$\Delta PV_d \approx \frac{\partial PV_d}{\partial S_0} \Delta S = \Delta_{FX} (S_0 \cdot \Delta S / S_0) = (S_0 \Delta_{FX}) \cdot \frac{\Delta S}{S_0}$$

So USD P&L per 1% FX move is:

$$\boxed{FX01 := 0.01 \cdot S_0 \cdot \Delta_{FX}}$$

---

### 2.4 Curve-Specific PV01 (Discount vs Projection)

The interest-rate-modeling text emphasizes that discounting and forecasting curves can be distinct, and it suggests measuring sensitivities by perturbing the market instruments that define those curves (funding instruments for discounting; index instruments for forecasting; basis instruments for basis risk).

For any curve object $x$, define:

$$\boxed{PV01_x := PV_d(\text{curve } x \text{ bumped by } +1\text{bp}) - PV_d}$$

**Bump Definition Warning:**

A "+1 bp" can mean:
- Parallel shift of continuously-compounded zero rates, or
- Parallel shift of par quotes with full rebuild, etc.

I'm not sure which one your desk uses; to be certain, we need your risk system's bump specification (zero bump vs par bump/rebuild, and which instruments are rebuilt).

---

### 2.5 Cross-Currency Basis Swap PV and Basis DV01

The interest-rate-modeling text gives the USD PV of a USD/JPY basis swap (receive USD Libor flat, pay JPY Libor + $e_{¥}$) as:

$$V_{\text{basisswap},\$}(0) = \sum_i L_{\$}(0, t_i, t_{i+1}) \tau_i P_{\$}(0, t_{i+1}) + P_{\$}(0, t_n)$$
$$\quad - X(0) \left( \sum_i (L_{¥}(0, t_i, t_{i+1}) + e_{¥}) \tau_i P_{¥}(0, t_{i+1}) + P_{¥}(0, t_n) \right)$$

and provides an equivalent rearrangement (equations (6.41)–(6.42)).

**Key Point for Basis DV01:**

The spread enters linearly as:

$$- X(0) \cdot e_{¥} \sum_i \tau_i P_{¥}(0, t_{i+1}) \quad \text{(inside the foreign-leg PV)}$$

Therefore, holding spot and curves fixed, the sensitivity to the quoted basis (per 1 unit of spread) is:

$$\frac{\partial V}{\partial e_{¥}} = - X(0) \sum_i \tau_i P_{¥}(0, t_{i+1})$$

and the basis DV01 is:

$$\boxed{PV01_{\text{basis}} = \frac{\partial V}{\partial e} \cdot 1\text{bp} = \left(\frac{\partial V}{\partial e}\right) \cdot 0.0001}$$

**Sanity Check:**

If you pay "foreign + basis", then increasing basis should make the swap worse (PV down) $\Rightarrow$ negative basis DV01. The formula matches that sign.

---

## Measurement & Risk (Chapter 31 Focus)

### A) PV in a Chosen Reporting Currency

**Define Reporting Currency:**

Let $d$ be the reporting (domestic) currency.

**PV of Multi-Currency Cashflows in Domestic Terms:**

With spot $S_0 = d/f$, a useful decomposition is:

$$PV_d = PV(\text{domestic cashflows discounted on } d\text{-curve}) \;+\; S_0 \cdot PV(\text{foreign cashflows discounted on } f\text{-curve})$$

This structure aligns with the text's arbitrage-consistency logic linking FX forwards and discounting across currencies.

**Clarify What Gets Discounted on Which Curve:**

- **Domestic cashflows:** Discount on domestic curve(s) in currency $d$
- **Foreign cashflows:** Discount on foreign curve(s) in currency $f$, then convert at spot $S_0$

If you use collateral-currency discounting, the appropriate discount curve may be driven by CSA collateral currency rather than payment currency. I'm not sure the correct discounting choice without your CSA/collateral currency and whether trades are cleared or bilateral.

---

### B) Risk Decomposition (First-Order)

#### FX Delta

**Definition:**

$$\Delta_{FX} := \frac{\partial PV_d}{\partial S_0}$$

Delta as a sensitivity concept is standard: $\Delta = \partial V / \partial X$.

**Units:**

If $S_0$ is $d/f$, then $\Delta_{FX}$ has units $f$.

Desk reports often also show:

$$FX01 = 0.01 \cdot S_0 \cdot \Delta_{FX} \quad \text{(units: } d \text{ per 1\% FX move)}$$

---

#### Rates PV01 by Currency (Discount and Projection Curves)

**Domestic Discount PV01:**

$$PV01_{d,\text{disc}} := PV_d(P_d \text{ bumped } +1\text{bp}) - PV_d$$

**Domestic Projection PV01 (if relevant):**

$$PV01_{d,\text{proj}} := PV_d(\text{projection curve for } d \text{ bumped } +1\text{bp}) - PV_d$$

with discount curve held fixed.

**Foreign Analogs:** $PV01_{f,\text{disc}}$, $PV01_{f,\text{proj}}$

**"What Does Bump Mean?"**

The interest-rate-modeling text motivates measuring sensitivities through perturbations to the market instruments used for curve construction (funding instruments, basis swaps, etc.).

Operationally, desks choose either:
- Parallel zero-rate bump (direct bump of curve nodes), or
- Par-quote bump + rebuild (bump market quotes, rebuild curves)

I'm not sure which you want without your system definition; it materially changes PV01 attribution.

---

#### Basis Risk (Cross-Currency Basis)

**Definition (Basis DV01):**

If the product PV depends on the quoted cross-currency basis spread $e$:

$$PV01_{\text{basis}} := PV_d(e + 1\text{bp}) - PV_d(e)$$

The text's explicit basis-swap valuation (USD/JPY) shows how $e$ enters linearly into PV.

**Distinct from FX Delta and Rate PV01:**

| Risk Bucket | Sensitivity To |
|-------------|----------------|
| FX delta | Spot $S_0$ |
| Rate PV01 | $P_d$ or $P_f$ (and/or projection curves) |
| Basis DV01 | Quoted spread $e$ that reconciles the cross-currency swap market |

---

### C) "What to Hedge with What?"

**Practitioner Mapping (First-Order Buckets):**

| Exposure | Hedge Instrument | Rationale |
|----------|------------------|-----------|
| FX delta | FX spot/forwards (or FX swap for rolling horizon/funding) | FX forward value changes with spot; delta is the leading exposure |
| Domestic rate PV01 | Domestic swaps/futures/bonds | These instruments are directly sensitive to the domestic curve you are bumping |
| Foreign rate PV01 | Foreign swaps/futures/bonds | These instruments load on the foreign curve; in domestic reporting, you will also manage their FX delta separately (often with FX forwards) |
| Basis DV01 | Cross-currency basis swaps (matching convention/tenor as closely as possible) | Basis DV01 is explicitly the derivative w.r.t. the quoted basis spread $e$ that appears in basis swaps |

---

### D) Why Curve Choice Matters for Hedges

Even if you treat curves as "inputs", hedge ratios depend on which curves you use.

**Discounting Curve Choice Changes PV and PV01:**

- OIS discounting is explicitly referenced as used in Hull's DerivaGem worksheets for certain interest-rate derivatives
- Hull's risk-management text notes that after the 2007–08 crisis, institutions switched from LIBOR/swap rates to OIS rates as proxies for risk-free rates

**Projection Curve Choice Changes Floating-Leg PV and PV01 Attribution:**

The interest-rate-modeling text describes separate discount and forward curves and defines forward rates from the index curve.

**Basis Definition/Sign/Curve Build Can Change Hedge Ratios:**

Basis swaps are quoted with a par spread $e_{\text{par}}$ that makes PV zero, but which leg the spread is attached to and sign conventions vary across markets.

I'm not sure your desk's exact quoting (receive/pay basis, which leg is "+basis"). To be certain, we need: currency pair, quote convention, and your risk system sign definition.

**Numerical Demonstration (Preview; Full in Example K):**

Changing domestic discount factor from $P_d(0,2) = 0.94$ to $0.945$ changes the fair forward $F_0 = S_0 P_f / P_d$, and therefore the locked-in domestic cash amount for an FX hedge changes. This directly changes PV and hedge ratios when you re-price or re-hedge.

---

### E) Keep It Focused

- We do **not** bootstrap cross-currency curves here; we treat curves and basis quotes as given inputs
- We do **not** do XVA; collateral currency matters only insofar as it selects discount curves and hence changes PV/PV01 and hedge ratios

---

## Worked Examples

### Shared Baseline Conventions for Examples A–I

| Parameter | Value |
|-----------|-------|
| Reporting currency $d$ | USD |
| Foreign currency $f$ | EUR |
| Spot $S_0$ | 1.10 USD/EUR |
| $P_d(0,1)$ | 0.97 |
| $P_d(0,2)$ | 0.94 |
| $P_f(0,1)$ | 0.98 |
| $P_f(0,2)$ | 0.95 |
| Bump definition | $P'(0,T) = P(0,T) \exp(-0.0001 \cdot T)$ |
| Day count/compounding | Avoided by working directly with discount factors |

---

### Example A: PV Conversion Sanity — Two Deterministic Cashflows

**Cashflows:**
- Receive +100 USD at $T = 1$
- Receive +80 EUR at $T = 2$

**PV in USD:**

$$PV = 100 \cdot 0.97 + 1.10 \cdot 80 \cdot 0.95$$

**Intermediate Steps:**

| Component | Calculation | Result |
|-----------|-------------|--------|
| Domestic PV | $100 \cdot 0.97$ | 97.00 USD |
| Foreign PV in EUR | $80 \cdot 0.95$ | 76.00 EUR |
| Convert to USD | $S_0 \cdot 76.00 = 1.10 \cdot 76.00$ | 83.60 USD |
| **Total** | $97.00 + 83.60$ | **180.60 USD** |

**Unit Checks:**
- $1.10$ USD/EUR $\times$ 76 EUR = USD ✓

---

### Example B: FX Delta via Finite Difference — Spot +1%

**Spot Bump:**

$$S_1 = 1.10 \times 1.01 = 1.111$$

**Reprice:**

$$PV(S_1) = 97.00 + 1.111 \cdot 76.00 = 97.00 + 84.436 = 181.436 \text{ USD}$$

**Finite-Difference FX Delta:**

| Quantity | Calculation | Result |
|----------|-------------|--------|
| $\Delta PV$ | $181.436 - 180.60$ | 0.836 USD |
| $\Delta S$ | $1.111 - 1.10$ | 0.011 USD/EUR |
| $\Delta_{FX}$ | $\Delta PV / \Delta S = 0.836 / 0.011$ | **76.00 EUR** |

**"PV per 1% Move":**

+1% move produced +0.836 USD, so $FX01 = +0.836$ USD per +1% move.

---

### Example C: Domestic PV01 — +1bp Bump to Domestic Discount Curve

**Domestic DF Bump at $T = 1$:**

$$P_d'(0,1) = 0.97 \exp(-0.0001) \approx 0.97 \cdot 0.9999 = 0.969903$$

**Reprice:**

$$PV' = 100 \cdot 0.969903 + 83.60 = 96.9903 + 83.60 = 180.5903 \text{ USD}$$

**PV01 (Definition: +1bp Bump):**

$$PV01_{d,\text{disc}} = PV' - PV = 180.5903 - 180.60 = \mathbf{-0.0097 \text{ USD/bp}}$$

**DV01 (Tuckman-Style):**

Tuckman defines $DV01 = P(y - 1\text{bp}) - P(y)$.

Under symmetric small bumps, $DV01 \approx -PV01$ in sign.

---

### Example D: Foreign PV01 — +1bp Bump to Foreign Discount Curve

**Foreign DF Bump at $T = 2$:**

$$P_f'(0,2) = 0.95 \exp(-0.0002) \approx 0.95 \cdot 0.9998 = 0.94981$$

**Reprice Foreign PV:**

| Step | Calculation | Result |
|------|-------------|--------|
| Foreign PV in EUR | $80 \cdot 0.94981$ | 75.9848 EUR |
| Convert to USD | $1.10 \cdot 75.9848$ | 83.58328 USD |

**Total PV':**

$$PV' = 97.00 + 83.58328 = 180.58328 \text{ USD}$$

**PV01:**

$$PV01_{f,\text{disc}} = 180.58328 - 180.60 = \mathbf{-0.01672 \text{ USD/bp}}$$

---

### Example E: Hedging FX Delta with Forwards — Neutralize FX Delta at $t = 0$

**Position to Hedge:** The EUR 80 receipt at $T = 2$ from Example A.

**Fair Forward Rate $K = F_{0,2}$ Using Discount Factors:**

$$K = S_0 \frac{P_f(0,2)}{P_d(0,2)} = 1.10 \cdot \frac{0.95}{0.94} = \mathbf{1.111702 \text{ USD/EUR}}$$

(Derived from Hull's parity form.)

**Hedge Trade:**

Short a 2Y forward to sell $N = 80$ EUR at $K = 1.111702$.

**FX Delta Check:**

Forward (short) has:

$$\Delta_{FX,\text{fwd}} = -N \cdot P_f(0,2) = -80 \cdot 0.95 = -76 \text{ EUR}$$

Original position has +76 EUR delta (Example B). **Net FX delta $\approx 0$**.

**Remaining Rate Exposures (Discount PV01s):**

*Domestic bump (+1 bp):*

$P_d(0,2) \to 0.94 e^{-0.0002} = 0.939812$

$$V_{\text{fwd}}' = 80(K \cdot 0.939812 - S_0 \cdot 0.95) = 80(1.04479088 - 1.045) = -0.01673 \text{ USD}$$

So $PV01_{d,\text{disc}}(\text{fwd}) \approx -0.01673$ USD/bp.

*Foreign bump (+1 bp):*

$P_f(0,2) \to 0.94981$

$$V_{\text{fwd}}' = 80(K \cdot 0.94 - S_0 \cdot 0.94981) = 80(1.04499988 - 1.044791) = +0.01671 \text{ USD}$$

So $PV01_{f,\text{disc}}(\text{fwd}) \approx +0.01671$ USD/bp.

**Interpretation:**

The FX hedge reduced FX delta but shifted curve PV01s—especially domestic PV01—because forwards embed discounting terms.

---

### Example F: Foreign Bond Hedged Back to Domestic — FX Reduced, Foreign PV01 Remains

**Foreign Bond (EUR):**

- Face: 100 EUR
- Coupon: 5% annual
- Maturity: 2Y
- Cashflows: 5 EUR at 1Y; 105 EUR at 2Y

**Bond PV in EUR:**

$$PV_f = 5 \cdot 0.98 + 105 \cdot 0.95 = 4.90 + 99.75 = 104.65 \text{ EUR}$$

**Unhedged PV in USD:**

$$PV_d = S_0 \cdot PV_f = 1.10 \cdot 104.65 = 115.115 \text{ USD}$$

**Unhedged FX Delta:**

$$\Delta_{FX} = PV_f = 104.65 \text{ EUR}$$

**Unhedged Foreign Discount PV01 (+1bp Bump to $P_f$):**

| Bumped Value | Calculation |
|--------------|-------------|
| $P_f(1)$ | $0.98 e^{-0.0001} = 0.979902$ |
| $P_f(2)$ | $0.95 e^{-0.0002} = 0.94981$ |

New EUR PV:

$$PV_f' = 5 \cdot 0.979902 + 105 \cdot 0.94981 = 4.89951 + 99.73005 = 104.62956$$

New USD PV: $PV_d' = 1.10 \cdot 104.62956 = 115.09252$

**Foreign PV01:** $PV_d' - PV_d = -0.02260$ USD/bp

**Hedge FX with a Single 2Y Forward (Imperfect Hedge):**

Short a forward to sell $N = 100$ EUR at 2Y, at fair $K = 1.111702$.

- Forward FX delta: $-N P_f(0,2) = -100 \cdot 0.95 = -95$ EUR
- **Net FX delta:** $104.65 - 95 = 9.65$ EUR (reduced, not zero)

**Foreign PV01 After Hedge:**

Forward foreign PV01 under +1bp foreign bump:

$$V_{\text{fwd}}' = 100(K \cdot 0.94 - S_0 \cdot 0.94981) = 100(1.04499988 - 1.044791) = +0.02089 \text{ USD}$$

**Total foreign PV01:** $-0.02260 + 0.02089 = -0.00171$ USD/bp (still nonzero)

**Basis Exposure Note:**

If your forward pricing uses curves incorporating cross-currency basis, the hedged PV will carry basis DV01. I'm not sure how to model this without your CSA/discounting framework and basis-quote convention.

---

### Example G: FX Swap vs Forward as Funding Hedge — Spot + Forward Decomposition

**Setup (Toy):**

| Parameter | Value |
|-----------|-------|
| $S_0$ | 1.10 USD/EUR |
| Horizon $T$ | 0.5y |
| $P_d(0, 0.5)$ | 0.985 (toy) |
| $P_f(0, 0.5)$ | 0.990 (toy) |

**Fair Forward:**

$$F_{0,0.5} = S_0 \frac{P_f}{P_d} = 1.10 \cdot \frac{0.990}{0.985} = 1.105584$$

**FX Swap as "Spot + Opposite Forward" (Conceptual):**

1. At $t = 0$: Exchange notionals at spot (buy EUR, sell USD)
2. At $T$: Reverse exchange at forward (sell EUR, buy USD)

**Quantify Forward Points / Carry:**

- Forward points: $F - S = 1.105584 - 1.10 = 0.005584$ USD/EUR
- For $N = 10,000,000$ EUR, forward-points difference in USD at maturity: $0.005584 \cdot N = 55,840$ USD

I'm not sure about exact spot/forward settlement conventions (T+2, tom/next, holiday calendars) because those are market- and pair-dependent and not specified in the sources you listed. To be certain, we need the currency pair and the market's standard settlement conventions.

---

### Example H: Cross-Currency Swap Exposure Breakdown — 2-Period Toy Basis Swap

**Structure (Adapted from the Book's Basis Swap PV Form):**

Receive USD floating + principal, pay EUR floating + $e$ + principal. The book provides the analogous USD/JPY valuation form.

**Inputs:**

| Parameter | Value |
|-----------|-------|
| $t_1$, $t_2$ | 1, 2 |
| $\tau_1$, $\tau_2$ | 1, 1 |
| $S_0$ | 1.10 |
| $P_d(1)$, $P_d(2)$ | 0.97, 0.94 |
| $P_f(1)$, $P_f(2)$ | 0.98, 0.95 |
| $P_f^{(L)}(1)$, $P_f^{(L)}(2)$ | 0.97902, 0.94810 (toy) |
| Basis spread $e$ | 0.0020 (20 bp) |

**EUR Forward Rates (from Projection Curve):**

$$L_f(0, 0, 1) = (1/0.97902 - 1) = 0.02143$$
$$L_f(0, 1, 2) = (0.97902/0.94810 - 1) = 0.03263$$

**USD Floating-Leg PV (Notional 1):**

Compute USD forwards from discount factors (since $P_d^{(L)} = P_d$ in the text's simplifying assumption):

$$L_d(0, 0, 1) = (1/0.97 - 1) = 0.0309278$$
$$L_d(0, 1, 2) = (0.97/0.94 - 1) = 0.0319149$$

PV:

$$PV_{USD} = L_d(0,0,1) \cdot 0.97 + L_d(0,1,2) \cdot 0.94 + 0.94 \approx 0.029999 + 0.029999 + 0.94 = 0.999998 \approx 1.0000$$

**EUR Leg PV in EUR (Notional 1):**

$$PV_{EUR} = (L_f(0,0,1) + e) \cdot 0.98 + (L_f(0,1,2) + e) \cdot 0.95 + 0.95$$

Intermediate:

| Period | Calculation | Result |
|--------|-------------|--------|
| Period 1 | $(0.02143 + 0.0020) \cdot 0.98 = 0.02343 \cdot 0.98$ | 0.0229614 |
| Period 2 | $(0.03263 + 0.0020) \cdot 0.95 = 0.03463 \cdot 0.95$ | 0.0328985 |
| Principal | | 0.95 |
| **Total** | | **1.0058599** |

**Swap PV in USD:**

$$PV = PV_{USD} - S_0 \cdot PV_{EUR} \approx 1.0000 - 1.10 \cdot 1.0058599 = \mathbf{-0.1064459}$$

**Risk Decomposition:**

*FX Delta:*

$$\Delta_{FX} = \frac{\partial PV}{\partial S_0} = -PV_{EUR} = -1.0058599 \text{ EUR}$$

*Basis DV01:*

$$\frac{\partial PV}{\partial e} = -S_0 (\tau_1 P_f(1) + \tau_2 P_f(2)) = -1.10(0.98 + 0.95) = -2.123$$

Per 1bp: $PV01_{\text{basis}} = -2.123 \cdot 0.0001 = -0.0002123$ USD/bp

*Domestic Discount PV01:*

Bump $P_d$, hold others fixed. $PV_{USD}$ changes from $\approx 1.0000$ to $\approx 0.9998005$, so:

$$PV01_{d,\text{disc}} \approx -0.0001995 \text{ USD/bp}$$

*Foreign Discount PV01:*

Bump $P_f$, hold others fixed. $PV_{EUR}$ falls slightly; $PV$ increases:

$$PV01_{f,\text{disc}} \approx +0.0002189 \text{ USD/bp}$$

---

### Example I: Basis Shock — Widen Basis by +5bp, Hold Spot and Curves Fixed

Using Example H:

$\Delta e = +5$ bp $= 0.0005$

**PV Change:**

$$\Delta PV \approx \frac{\partial PV}{\partial e} \cdot \Delta e = (-2.123)(0.0005) = -0.0010615 \text{ USD}$$

**Basis DV01 (per 1bp) from Finite Difference:**

$$\frac{\Delta PV}{5} = -0.0002123 \text{ USD/bp}$$

matching the analytic basis DV01.

---

### Example J: Hedge Mapping Demo — Stepwise Hedging in Practice (Toy Risk Numbers)

**Conventions:**

- Reporting currency: USD
- Spot quote: $S$ = USD per EUR
- Risk buckets: FX delta (EUR), domestic PV01 (USD/bp), foreign PV01 (USD/bp), basis DV01 (USD/bp)

Hedge-instrument sensitivities are assumed (as would come from your risk system). I'm not sure what the correct numbers are without your curves, instruments, CSA, and risk methodology; this is a pedagogical hedge-mapping demo.

**Initial Portfolio Exposures:**

| Risk Bucket | Exposure |
|-------------|----------|
| FX delta | +50,000,000 EUR |
| Domestic PV01 | −220,000 USD/bp |
| Foreign PV01 | −180,000 USD/bp |
| Basis DV01 | +35,000 USD/bp |

**Hedge Instruments (Assumed Linear Sensitivities):**

| Instrument | Sensitivity |
|------------|-------------|
| FX forward (sell EUR) | FX delta $\approx -0.99$ EUR per 1 EUR notional |
| Domestic swap | PV01 = +85,000 USD/bp per 100mm USD notional |
| Foreign swap | PV01 = +70,000 USD/bp per 100mm EUR notional (in USD reporting) |
| Basis swap | Basis DV01 = −40,000 USD/bp per 100mm USD notional |

---

#### Step 1: Hedge FX Delta (Forwards)

Need forward notional $N_{FX}$ such that:

$$-0.99 \cdot N_{FX} = -50{,}000{,}000 \quad \Rightarrow \quad N_{FX} = \frac{50{,}000{,}000}{0.99} = 50{,}505{,}050.5 \text{ EUR}$$

**After Step 1:**

| Risk Bucket | Exposure |
|-------------|----------|
| FX delta | $\approx 0$ EUR |
| Domestic PV01 | −220,000 USD/bp |
| Foreign PV01 | −180,000 USD/bp |
| Basis DV01 | +35,000 USD/bp |

(In reality, the FX forward also carries rate PV01; see Example E.)

---

#### Step 2: Hedge Domestic PV01 (Domestic Swaps/Futures/Bonds)

Need domestic swap notional $N_d$ (in 100mm blocks) such that:

$$(+85{,}000) \cdot \left(\frac{N_d}{100\text{mm}}\right) = 220{,}000 \quad \Rightarrow \quad \frac{N_d}{100\text{mm}} = \frac{220}{85} = 2.588235$$

So $N_d \approx 258.8235$ mm USD.

**After Step 2:**

| Risk Bucket | Exposure |
|-------------|----------|
| FX delta | $\approx 0$ |
| Domestic PV01 | $\approx 0$ |
| Foreign PV01 | −180,000 |
| Basis DV01 | +35,000 |

---

#### Step 3: Hedge Foreign PV01 (Foreign Swaps/Futures/Bonds)

Need foreign swap notional $N_f$ (in 100mm blocks) such that:

$$(+70{,}000) \cdot \left(\frac{N_f}{100\text{mm}}\right) = 180{,}000 \quad \Rightarrow \quad \frac{N_f}{100\text{mm}} = \frac{180}{70} = 2.571429$$

So $N_f \approx 257.1429$ mm EUR.

**After Step 3:**

| Risk Bucket | Exposure |
|-------------|----------|
| FX delta | $\approx 0$ (by bucket assumption; in a full system you may need to re-trim FX) |
| Domestic PV01 | $\approx 0$ |
| Foreign PV01 | $\approx 0$ |
| Basis DV01 | +35,000 |

---

#### Step 4: Hedge Basis DV01 (Cross-Currency Basis Swap)

Need basis-swap notional $N_b$ (in 100mm blocks) such that:

$$(-40{,}000) \cdot \left(\frac{N_b}{100\text{mm}}\right) = -35{,}000 \quad \Rightarrow \quad \frac{N_b}{100\text{mm}} = \frac{35}{40} = 0.875$$

So $N_b = 87.5$ mm USD in the direction that produces −40,000 USD/bp per 100mm.

**After Step 4:**

| Risk Bucket | Exposure |
|-------------|----------|
| FX delta | $\approx 0$ |
| Domestic PV01 | $\approx 0$ |
| Foreign PV01 | $\approx 0$ |
| Basis DV01 | $\approx 0$ |

**Interpretation:**

This is the desk workflow: hedge the largest/most liquid bucket first (often FX), then rates, then basis. In practice you iterate because hedges can create cross-exposures (Example E).

---

### Example K: Curve-Choice Sensitivity — PV and Hedge Ratios Change Under Different Discount Curves

**Goal:** Show that changing the discounting curve changes PV and forward rates.

**Conventions:**

- Same as Example E (hedging EUR 80 at 2Y)
- Spot $S_0 = 1.10$, foreign DF $P_f(0,2) = 0.95$
- Two domestic discount-curve choices:
  - Curve A: $P_d^A(0,2) = 0.94$
  - Curve B: $P_d^B(0,2) = 0.945$

This is a sensitivity experiment. I'm not sure which curve is "correct" without the CSA/collateral currency and desk discounting policy. Modern practice often uses OIS-type discounting in derivatives contexts (noted in Hull materials).

**Step 1: Compute the Fair Forwards**

$$K_A = S_0 \frac{P_f}{P_d^A} = 1.10 \cdot \frac{0.95}{0.94} = 1.111702$$

$$K_B = S_0 \frac{P_f}{P_d^B} = 1.10 \cdot \frac{0.95}{0.945} = 1.105820$$

**Step 2: Compare Locked-In Domestic Cash Amounts for Hedging 80 EUR**

| Curve | Locked USD at Maturity | Calculation |
|-------|------------------------|-------------|
| Curve A | 88.93616 USD | $80 \cdot 1.111702$ |
| Curve B | 88.46560 USD | $80 \cdot 1.105820$ |
| **Difference** | **0.47056 USD** | |

**Step 3: Interpret**

A different discount curve implies a different "fair" forward, hence a different FX hedge rate and potentially different hedge ratios in a system that re-optimizes hedges.

Even if PV at inception is forced to 0 for the forward (by using the fair $K$), the risk (PV01 attribution, basis attribution, etc.) depends on the curve framework.

---

### Example L: Scenario P&L Explain — Unhedged vs Hedged, 4 Scenarios, Factor P&L Table

Use Example J exposures.

**Base Spot:** $S_0 = 1.10$ USD/EUR

**Compute FX01 for the Unhedged Portfolio:**

$$FX01 = 0.01 \cdot S_0 \cdot \Delta_{FX} = 0.01 \cdot 1.10 \cdot 50{,}000{,}000 = 550{,}000 \text{ USD per +1\%}$$

**Scenarios:**

1. Spot FX move: +2% in $S$
2. Domestic curve: +25 bp parallel shift
3. Foreign curve: +25 bp parallel shift
4. Basis: +10 bp widening

Assume the hedged portfolio after Example J has bucket exposures $\approx 0$ (ignoring second-order/cross effects).

| Scenario | Shock | Unhedged P&L (USD) | Hedged P&L (USD) | Dominant Bucket |
|----------|-------|---------------------|------------------|-----------------|
| 1 | $S$ +2% | $2 \times 550{,}000 = +1{,}100{,}000$ | $\approx 0$ | FX delta |
| 2 | $d$-curve +25bp | $25 \times (-220{,}000) = -5{,}500{,}000$ | $\approx 0$ | Domestic PV01 |
| 3 | $f$-curve +25bp | $25 \times (-180{,}000) = -4{,}500{,}000$ | $\approx 0$ | Foreign PV01 |
| 4 | basis +10bp | $10 \times (+35{,}000) = +350{,}000$ | $\approx 0$ | Basis DV01 |

**Explain Which Hedge Worked:**

- **Scenario 1:** FX forward hedge neutralized FX delta
- **Scenario 2:** Domestic swap hedge neutralized domestic PV01
- **Scenario 3:** Foreign swap hedge neutralized foreign PV01
- **Scenario 4:** Basis swap hedge neutralized basis DV01

**Reality Check:**

In a full repricing system, each hedge can create secondary exposures (e.g., FX forward creates rate PV01; see Example E). Therefore "$\approx 0$" becomes "small but nonzero," and you iterate.

---

## Practical Notes

### Risk Report Glossary (Must Be Consistent)

**FX Delta:**
- Definition: $\Delta_{FX} = \partial PV_d / \partial S$
- Units if $S = d/f$: foreign currency $f$
- Common desk conversion: $FX01 = 0.01 \cdot S \cdot \Delta_{FX}$ (units: domestic currency $d$ per 1% move)

**PV01 by Curve and by Currency:**
- $PV01_{d,\text{disc}} = PV(\text{domestic discount curve } +1\text{bp}) - PV$ in $d$/bp
- $PV01_{d,\text{proj}} = PV(\text{domestic projection curve } +1\text{bp}) - PV$
- Analogous for $f$

**Basis DV01:**
- $PV01_{\text{basis}} = PV(e + 1\text{bp}) - PV(e)$ in $d$/bp
- Sign convention must be stated (pay vs receive basis; which leg has "+$e$")

### Common Pitfalls

1. **Mixing up quote direction** $S$ vs $1/S$ and flipping FX delta sign (Hull explicitly warns on quote conventions)
2. **Hedging FX delta but forgetting foreign rate PV01** (especially for foreign-currency bonds and swaps)
3. **Assuming a "currency hedge" eliminates all currency-related risk:** basis risk remains for cross-currency structures
4. **Inconsistent curve choices** between pricing and risk (wrong hedge ratios)
5. **Confusing discount PV01 vs projection PV01** in a multi-curve framework

### Implementation Pitfalls

- Using spot vs forward inconsistently for PV conversion (spot is used for immediate conversion; forwards govern future exchanges)
- Mixing bump-and-rebuild vs direct zero bumps across systems (PV01 mismatch)
- Calendar/settlement mismatches in FX forwards and FX swaps: I'm not sure the correct rules without the currency pair and market conventions

### Verification Tests

1. **Unit checks:** Every PV term must end in reporting currency $d$
2. **FX hedge sanity:** After FX hedge, PV sensitivity to spot is materially reduced (Example E)
3. **Small-bump stability:** PV01 and basis DV01 stable under 0.5bp vs 1bp bumps
4. **Repricing checks:** All hedge instruments are re-priced on the same curve framework used for the portfolio

---

## Summary & Recall

### 10-Bullet Executive Summary

1. **Choose a reporting currency $d$**; state it explicitly
2. **Make FX quote direction explicit:** $S = d/f$ or $S = f/d$; deltas flip under inversion
3. **PV in domestic currency** can be decomposed into domestic PV plus spot-times-foreign PV
4. **FX delta** is $\partial PV_d / \partial S$ (units: foreign currency if $S = d/f$)
5. **Rate risk splits** by currency and (in multi-curve) by discount vs projection curves
6. **PV01 depends on the bump definition** (zero bump vs par bump/rebuild). Specify it.
7. **Cross-currency basis swaps** embed a quoted spread $e$; basis DV01 is $\partial PV / \partial e \cdot 1$bp
8. **"What to hedge with what":** FX delta ↔ forwards; domestic PV01 ↔ domestic swaps/bonds; foreign PV01 ↔ foreign swaps/bonds; basis DV01 ↔ basis swaps
9. **Curve choice** (OIS vs other, collateral currency) changes PV and hedge ratios; specify your curve framework
10. **Hedge programs are iterative** because hedges can create secondary exposures (Example E)

### Cheat Sheet of Formulas

**PV in Domestic Currency:**

$$PV_d = \sum_k C_{d,k} P_d(0, T_k) + S_0 \sum_j C_{f,j} P_f(0, T_j)$$

**FX Delta:**

$$\Delta_{FX} = \frac{\partial PV_d}{\partial S_0}$$

**FX01 (USD per 1% Move):**

$$FX01 = 0.01 \cdot S_0 \cdot \Delta_{FX}$$

**PV01 to a Curve $x$:**

$$PV01_x = PV(\text{curve } x \text{ bumped } +1\text{bp}) - PV$$

**Basis DV01 (from Basis Swap PV Linearity in $e$):**

$$PV01_{\text{basis}} = \frac{\partial PV}{\partial e} \cdot 0.0001$$

**Forward FX (Hull):**

$$F_0 = S_0 e^{(r_d - r_f)T}$$

---

### Flashcards (35 Q/A Pairs)

| # | Question | Answer |
|---|----------|--------|
| 1 | What must be specified first in multi-currency PV? | The reporting currency $d$ and FX quote direction $S$ |
| 2 | If $S = d/f$, what are the units of FX delta $\partial PV_d / \partial S$? | Units of $f$ |
| 3 | What is $FX01$? | Approx. USD P&L for a +1% move in spot: $0.01 \cdot S \cdot \Delta_{FX}$ |
| 4 | Why can FX hedges change rate PV01? | FX forwards embed discounting terms, creating rate sensitivities (Example E) |
| 5 | Define PV01 | PV(bumped +1bp) − PV (must specify bump definition) |
| 6 | How does Tuckman define DV01? | $DV01 = P(y - 1\text{bp}) - P(y)$ |
| 7 | Discount curve vs projection curve? | Discount curve values cashflows; projection curve generates forwards for float legs |
| 8 | What is basis DV01? | PV sensitivity to a 1bp move in the quoted cross-currency basis spread $e$ |
| 9 | In the book's USD/JPY basis swap, which leg has the spread $e_{¥}$? | JPY leg (pay JPY Libor + $e_{¥}$) when receiving USD Libor flat |
| 10 | What makes a basis quote "par"? | The value of $e$ such that basis swap PV is zero |
| 11 | What is a common pitfall with FX quotes? | Confusing $S$ with $1/S$ and flipping delta |
| 12 | What hedges FX delta? | FX spot/forwards (or FX swaps depending on horizon) |
| 13 | What hedges domestic PV01? | Domestic swaps/futures/bonds |
| 14 | What hedges foreign PV01? | Foreign swaps/futures/bonds (plus FX hedges as needed) |
| 15 | What hedges basis DV01? | Cross-currency basis swaps (matching convention) |
| 16 | What does "multi-curve" mean? | Separate curves for discounting and forecasting |
| 17 | Why does curve choice matter? | It changes PV and hedge ratios (Example K) |
| 18 | What must be specified to interpret PV01 numbers? | The bump method (zero bump vs rebuild; parallel vs key-rate) |
| 19 | What is the domestic PV contribution of a domestic cashflow $C_d$ at $T$? | $C_d P_d(0,T)$ |
| 20 | What is the domestic PV contribution of a foreign cashflow $C_f$ at $T$ under spot conversion? | $S_0 C_f P_f(0,T)$ |
| 21 | In a simple deterministic setting, what is $\Delta_{FX}$ for foreign cashflows? | $\sum C_f P_f$ |
| 22 | If you perfectly hedge each foreign cashflow with matching forwards, what happens to FX delta? | It is neutralized at $t = 0$ |
| 23 | Does FX hedging always eliminate foreign PV01? | Not necessarily; depends on hedge horizon/cashflow matching (Example F) |
| 24 | What's the key difference between rate PV01 and basis DV01? | Rate PV01 bumps curves; basis DV01 bumps the quoted spread $e$ |
| 25 | What is the "cross-currency yield spread" $s(t)$ used for? | Relating index/pseudo-discount to discount curve: $P^{(L)} = P e^{-st}$ |
| 26 | Name a verification test for FX delta hedging | PV sensitivity to spot materially reduced after hedge |
| 27 | What is a common mistake in multi-currency risk reports? | Reporting PV01s without specifying curve and bump definition |
| 28 | What is the most important "first step" before hedging? | Define factor buckets and units consistently |
| 29 | Why is basis risk "currency-related" even if FX delta is hedged? | Because basis moves change PV even at fixed spot and curves |
| 30 | What does it mean to "bump-and-rebuild"? | Bump market quotes, rebuild curves, then reprice |
| 31 | Why can bump-and-rebuild differ from direct zero bumps? | Rebuild changes the entire curve shape and projection-discount interactions |
| 32 | What is "par basis" in the text? | Basis spread making cross-currency basis swap PV zero |
| 33 | How do you convert FX delta to USD per 1% move? | Multiply by $0.01 \cdot S$ |
| 34 | What extra info is needed to choose the right discount curve? | CSA collateral currency, clearing status, desk policy (OIS vs other) |
| 35 | Why is iterative hedging common? | Hedges can introduce secondary exposures (FX hedges create PV01) |

---

## Mini Problem Set (18 Questions)

1. **Define $S$ and explain how FX delta changes if the market quotes $1/S$.**

   *Sketch:* Use chain rule: $\partial PV / \partial(1/S) = -S^2 \partial PV / \partial S$.

2. **For deterministic foreign cashflows, derive $\Delta_{FX} = \sum C_f P_f$.**

   *Sketch:* Differentiate $PV_d = \sum C_d P_d + S \sum C_f P_f$.

3. **In Example A, compute FX01 (USD P&L per +1% in spot).**

   *Sketch:* $FX01 = 0.01 \cdot S \cdot \Delta_{FX} = 0.01 \cdot 1.10 \cdot 76 = 0.836$.

4. **Using Tuckman's DV01 definition, explain sign differences between DV01 and PV01.**

   *Sketch:* DV01 uses yield down 1bp; PV01 as defined here uses yield up 1bp.

5. **In Example E, show that the FX hedge introduced domestic PV01.**

   *Sketch:* Reprice forward under bumped domestic DF; forward value changes with $P_d$.

6. **For a basis swap PV that includes $-S e \sum \tau P_f$, derive basis DV01.**

   *Sketch:* Differentiate w.r.t $e$, multiply by 1bp.

7. **Explain why discount vs projection PV01 can differ in sign for a floating leg.**

   *Sketch:* Discounting reduces PV; projection affects expected coupons; can offset.

8. **Explain what market instruments you would perturb to measure discounting vs forecasting vs basis risk.**

   *Sketch:* Funding instruments → discount; index instruments → forecasting; basis swaps → basis.

9. **Given a portfolio's FX delta in EUR and spot $S$, convert it to USD per 1% move.**

   *Sketch:* Multiply by $0.01 \cdot S$.

10. **Build a 3-cashflow multi-currency PV and compute FX delta analytically.**

11. **Create a key-rate bump PV01 grid for domestic and foreign curves (conceptually).**

12. **Show how a single-maturity FX forward hedge differs from a full cashflow-matched forward strip.**

13. **For a two-currency swap, explain qualitatively how collateral currency can change discounting.**

14. **Using Example H structure, compute the par basis $e_{\text{par}}$ that makes PV zero (solve for $e$).**

15. **Propose a hedge set when basis swaps are illiquid at the exact maturity (proxy hedging).**

16. **Describe how to validate that basis DV01 is stable across bump sizes (0.5bp vs 1bp).**

17. **Explain how to avoid double-counting FX effects when reporting PV01 in domestic currency.**

18. **Outline a P&L explain workflow that attributes moves to FX, domestic curve, foreign curve, and basis.**

---

## Source Map

### (A) Verified Facts — Directly Source-Backed

| Content | Source |
|---------|--------|
| FX spot/forward quote direction warning and forward formula $F_0 = S_0 e^{(r_d - r_f)T}$ | Hull |
| Delta as $\partial V / \partial X$ sensitivity concept | Hull |
| Multi-curve framing: separation of discount and forward curves; forwards derived from index curve | Andersen & Piterbarg |
| Sensitivity framing: perturbations to funding/index/basis instruments correspond to discounting/forecasting/basis sensitivities | Andersen & Piterbarg |
| Cross-currency basis swap PV form and definition of par basis $e_{\text{par}}$ | Andersen & Piterbarg |
| Cross-currency yield spread definition $P^{(L)} = P e^{-st}$ | Andersen & Piterbarg |
| DV01 finite-difference definition | Tuckman |
| Discount-factor curve definition as $P(t,T)$ "zero-bond curve" | Tuckman |
| Notes pointing to OIS discounting usage and post-crisis OIS shift | Hull |

### (B) Reasoned Inference — Derived in This Chapter

| Content | Derivation Logic |
|---------|------------------|
| Generic domestic-currency PV decomposition $PV_d = \sum C_d P_d + S \sum C_f P_f$ | Natural generalization of discount-factor valuation + FX conversion logic |
| FX01 conversion $FX01 = 0.01 \cdot S \cdot \Delta_{FX}$ | Algebra from delta definition |
| Basis DV01 formula | Derivative of the basis-swap PV expression with respect to $e$ |
| Hedge mapping "what to hedge with what" | Practitioner interpretation of the risk buckets |

### (C) Speculation — Explicitly Flagged Uncertainties

| Topic | Note |
|-------|------|
| Cross-currency basis quoting/sign conventions | I'm not sure without the currency pair, trade direction, and desk conventions |
| Collateral currency/CSA-driven discounting choice | I'm not sure without CSA and clearing details |
| Exact FX swap settlement/calendar conventions | I'm not sure without pair/market specification |
