# Chapter 28: Basis Trades in Rates — OIS-IBOR Basis, Treasury Futures Basis, Swap Spread RV, Curve RV

---

## Introduction

Prerequisites: [Chapter 09 — Repo and the Funding Engine](chapters/chapter_09_repo_funding_engine.md), [Chapter 10 — Treasuries Microstructure and Relative Value](chapters/chapter_10_treasury_microstructure_relative_value.md), [Chapter 18 — OIS Discounting Curve](chapters/chapter_18_ois_discounting_curve.md), [Chapter 19 — Projection Curves $LIBOR/SOFR$ and Multi-Curve](chapters/chapter_19_projection_curves_libor_sofr_multi_curve.md), [Chapter 20 — Tenor Basis](chapters/chapter_20_tenor_basis.md), [Chapter 23 — Treasury Futures](chapters/chapter_23_treasury_futures.md), [Chapter 26 — Swap PV01, DV01, and Hedging with Swaps](chapters/chapter_26_swap_pv01_dv01_hedging.md), [Chapter 27 — Swap Spreads, Asset Swaps, and Swap-Curve Relative Value](chapters/chapter_27_swap_spreads_asset_swaps_swap_curve_rv.md)  
Follow-on: [Chapter 29 — FX Spot and Forwards](chapters/chapter_29_fx_spot_forwards.md), [Chapter 30 — FX Swaps and Cross-Currency Swaps](chapters/chapter_30_fx_swaps_cross_currency_swaps.md), [Chapter 31 — Multi-Currency Risk](chapters/chapter_31_multi_currency_risk.md), [Chapter 32 — Counterparty Exposure Basics](chapters/chapter_32_counterparty_exposure_basics.md), [Chapter 33 — Collateral Discounting and OIS](chapters/chapter_33_collateral_discounting_ois.md), [Chapter 34 — XVA Overview](chapters/chapter_34_xva_overview.md)

## Learning Objectives
- Decompose a rates RV/basis idea into level risk, spread risk, and residual funding/liquidity risk.
- Translate common basis quotes into the cashflow/rate objects that drive PV.
- Compute and interpret risk measures with explicit bump object, bump size (`1 bp = 1e-4`), units, and sign.
- Explain why “convergence” is not arbitrage when leverage, margining, and liquidity constraints matter.

In 1998, Long-Term Capital Management ran leveraged convergence and relative-value trades designed to be "rate neutral" to first order. A toy example of the archetype is long one instrument and short a closely related instrument to earn a small spread (a *basis*)—then lever it up. When Russia defaulted and markets panicked, the residual risks (liquidity premia, funding, margin) moved violently, and the positions became catastrophically unprofitable. The lesson was not that the trade was wrong in some fundamental sense—the spread eventually narrowed—but that the *path* and the *financing* can kill you long before "convergence" arrives.

Every basis trade is, at its core, a bet on the difference between two similar-looking instruments. You are long one rate and short another. The instruments appear related—they might share the same maturity, reference the same credit quality, or track the same economic variable—but they are not identical. The spread between them is the *basis*, and movements in this basis determine your profit or loss.

This sounds simple, but basis trades are among the most treacherous in fixed income. A trader who believes they have isolated a clean relative value opportunity often discovers, painfully, that they are exposed to risks they never intended to take. A swap spread position, for example, is rarely “just credit vs risk-free”: it also embeds Treasury financing, benchmark (on-the-run) dynamics, regulatory/capital effects, and convexity-hedging flows.

This chapter develops a systematic framework for analyzing basis trades in rates:

1. **The "Long/Short" Decomposition** (Section 28.1): How to break any trade into its fundamental exposures before entering
2. **The Swap Spread Trade** (Section 28.2): What it really bets on, and why it's more complex than "credit vs. risk-free"
3. **The OIS-IBOR Basis** (Section 28.3): The discounting vs. projection spread and how the multi-curve framework creates new basis exposures, including detailed mathematical foundations
4. **Treasury Futures Basis** (Section 28.4): Gross/net basis, CTD optionality, and P&L mechanics—"the basis trade" that hedge funds run
5. **Curve Relative Value** (Section 28.5): Trading the "spread of spreads" across the term structure
6. **What Can Go Wrong: Case Studies** (Section 28.6): LTCM 1998 and the funding/margin mechanics that blow up basis trades
7. **Hidden Risks and Practical Execution** (Section 28.7): Funding, roll, liquidity, convexity, and the exposures traders forget
8. **Extended Worked Examples** (Section 28.8): Twelve comprehensive numerical examples covering multi-curve valuation, basis swap pricing, DV01 hedging, and curve trades

The material builds directly on Chapter 20's multi-curve framework, Chapter 23's Treasury futures mechanics, and Chapter 27's asset swap constructions. This chapter focuses not on the *definition* of these spreads but on *how traders use them*—and what can go wrong.

---

## 28.1 The "What You're Long / What You're Short" Framework

### 28.1.1 Every Trade Has a Decomposition

Before entering any basis position, a trader should be able to articulate precisely what they are buying and what they are selling. This sounds obvious, yet the failure to decompose trades properly has destroyed more P&L than almost any other analytical failure in fixed income.

A useful identity in Treasury futures basis trades is: the P&L of a long-basis position is approximately position size times the change in **net basis**. But net basis moves for many reasons; the formula tells you how to compute P&L, while decomposition tells you *why* you will make or lose money.

Consider a trader who wants to "buy the swap spread"—that is, bet that swap rates will fall relative to Treasury yields. The naive decomposition is simple: long credit risk, short risk-free rates. But this misses critical details:

- Which Treasury are you shorting? On-the-run or off-the-run?
- How are you financing the short Treasury position?
- What happens when the on-the-run Treasury "rolls off" and a new issue becomes the benchmark?
- Is the swap collateralized? If so, what currency is the collateral?

Each of these questions reveals a hidden exposure that can overwhelm the primary trade thesis.

### 28.1.2 A Framework for Decomposition

For any rates basis trade, there are typically three layers of exposure:

| Layer | What It Represents | Examples |
|-------|-------------------|----------|
| **Level Risk** | Directional exposure to the overall level of rates | Unhedged DV01, curve parallel shift |
| **Spread Risk** | Exposure to the spread between two rate curves | Swap spread, OIS-IBOR basis, tenor basis |
| **Residual Risk** | Exposures that remain after hedging | Funding, roll, convexity, liquidity, positioning |

A desk-friendly refinement is to split “spread risk” into three questions you can answer in a trade write-up:
- **Discounting basis:** which curve discounts your cashflows $and under what collateral/funding assumption$?
- **Projection basis:** which forward index/curve sets each floating coupon (and what basis quotes link it to other curves)?
- **Liquidity/funding component:** what financing rate $repo/term funding/haircuts/margin$ you are implicitly long/short?

A well-structured basis trade should have *zero* level risk $be DV01-neutral$, *intentional* spread risk (the view being expressed), and *minimized* residual risk (or at least, understood and accepted residual risk). The decomposition process forces the trader to verify that each layer is appropriately managed.

### 28.1.3 The Decomposition Process

**Step 1: Identify the Primary Exposure**

What spread are you trading? Be precise about the mathematical definition:
- Swap spread = Par swap rate − Treasury yield (but which Treasury?)
- OIS-IBOR basis = Term rate projection − OIS rate for equivalent tenor
- Treasury futures basis = Bond forward price − (Conversion factor × Futures price)

**Step 2: Identify the Hedge Instruments**

What instruments are you using to isolate the spread?
- A swap spread trade might use Treasuries hedged with swaps
- A tenor basis trade uses basis swaps (see Chapter 20)
- A futures basis trade uses Treasury bonds and Treasury futures

**Step 3: Decompose the Hedge Imperfections**

Even a "DV01-neutral" hedge is never perfect. For example, if your DV01 hedge is wrong, you will make or lose money on a 1bp parallel move in the hedging rate object even if the basis itself does not move.

The residual exposures include:
- **Curve shape risk**: Steepening/flattening affects bonds and swaps differently
- **Funding risk**: Financing the position costs money; rates can change
- **Roll/carry**: The trade ages; time decay and curve roll matter
- **Convexity differences**: Second-order effects diverge in large rate moves
- **Liquidity/specialness**: Specific instruments may cheapen or richen unexpectedly

**Step 4: Quantify the Residuals**

The critical question: If my primary view is correct (the spread moves as expected), but each residual risk moves against me by a reasonable amount, do I still make money? If the answer is "maybe not," the trade is poorly structured.

### 28.1.4 Risk Language: Bump Object, Units, and Sign

Basis trades are “small-spread, big-notional” positions, so tiny convention mismatches can swamp the intended view. Throughout this chapter, every risk number should come with four labels:

- **Bump object:** what is actually perturbed (a yield, zero curve nodes, par quotes with rebuild, a quoted basis spread, etc.).
- **Bump size:** `1 bp = 1e-4`.
- **Units:** currency per 1 bp per stated notional (e.g., “\$ per 1 bp per \$1mm”).
- **Sign convention:** this book uses **$DV01 := PV(\text{rates down }1\text{ bp}) - PV(\text{base})$** for the stated bump object.

For spread/basis risks, use the same discipline. If a quoted spread $e$ enters PV linearly (e.g., you pay “OIS + $e$”):
- A natural “basis PV01” is $PV(e\downarrow 1\text{ bp}) - PV(\text{base})$ in the same units.
- If the spread enters through curve construction (bump quote → rebuild curve → reprice), the bump object is the **market quote** and the rebuild rule must be stated.

A quick sanity check: if you cannot answer “what exactly did we bump?”, you do not yet know what your DV01/PV01 number means.

---

## 28.2 The Swap Spread Trade: Anatomy and Hidden Complexity

### 28.2.1 What Is a Swap Spread?

A swap spread is the difference between a swap rate and a government bond yield at (roughly) the same maturity. A common quote is the **on-the-run** Treasury yield versus the par swap rate.

The important desk caveat is that a quoted swap spread is not a “pure credit spread.” It is a spread between two instruments with different liquidity, financing, and optionality features, so its movements can mix multiple effects.

### 28.2.2 What Actually Drives Swap Spreads?

Swap spreads can move for multiple reasons:

**1. Funding / credit conditions**

Historically, term reference rates embedded bank term-funding premia. Even with RFR-based swaps, balance-sheet and funding conditions still affect swap pricing.

**2. Treasury supply/demand and liquidity premia**

The Treasury leg (especially if it is an on-the-run issue) can embed liquidity premia that have nothing to do with swap-market credit.

**3. Repo financing (“special” vs GC)**

If the benchmark Treasury is expensive to borrow (trades “special”), the economics of shorting it can change materially. Financing can dominate the P&L of a “swap spread” package even if the spread view is correct.

**4. Balance-sheet / regulatory constraints**

Changes in balance-sheet costs can shift demand between swaps and cash Treasuries, moving the spread.

**5. Convexity-hedging flows (e.g., mortgages)**

Large convexity-hedging flows can create “directionality” in swap spreads: widening in selloffs and narrowing in rallies, for reasons unrelated to credit.

### 28.2.3 The "Classic" Swap Spread Trade

**Trade**: Receive fixed on a swap, short the corresponding maturity Treasury

**Intended Bet**: Swap spreads will narrow (swap rate will fall relative to Treasury yield)

**Execution**:
- Receive fixed on \$100MM 10Y swap at, say, 4.50%
- Short $X$ face of 10Y Treasury (sized to match DV01)
- Fund the short Treasury position via repo

**What You're Long**:
- Treasury yield rising relative to swap rate
- "Credit spread narrowing" (loosely speaking)

**What You're Short**:
- Swap rate falling relative to Treasury yield
- "Credit spread widening"

### 28.2.4 Hidden Exposures in the Swap Spread Trade

Quoted swap spreads are useful for broad themes, but trade-level P&L can be dominated by details of benchmark choice, repo financing, and hedge drift.

**Exposure 1: On-the-Run / Off-the-Run Dynamics**

The swap spread is typically quoted against the *on-the-run* Treasury. On-the-run yields can embed liquidity and special-financing effects, so the quoted spread can be a noisy measure of the underlying relative value you think you are trading.

If you short the on-the-run 10Y to trade swap spreads, you are also implicitly long the "on-the-run premium." When that premium compresses (as the bond ages off-the-run and a new issue takes its place), you may lose money even if your credit view is correct.

**Exposure 2: Repo Financing**

Shorting the Treasury requires borrowing it in repo. If the bond goes "special" (trades at a repo rate well below general collateral), your financing cost increases. In stressed or crowded conditions, specials can become large enough that financing, not spread convergence, dominates realized P&L.

**Exposure 3: DV01 Mismatch Over Time**

Swap DV01 and Treasury DV01 drift differently as rates move and time passes. A trade that starts DV01-neutral will not remain so, requiring periodic rebalancing.

**Exposure 4: Convexity Mismatch**

Treasuries and swaps have different convexity profiles. In large rate moves, these second-order effects can generate significant unexpected P&L.

### 28.2.5 Better Ways to Trade Swap Spreads

To avoid contamination by on-the-run dynamics, practitioners often:

1. **Use Off-the-Run Treasuries**: Trade the spread to a seasoned Treasury, accepting less liquidity but cleaner economics
2. **Trade Asset Swaps**: Use the asset swap structure (Chapter 27) to isolate the credit component more precisely
3. **Trade "Adjusted" Swap Spreads**: Explicitly adjust for financing advantages and liquidity premia (recognizing that the adjustment itself is model-dependent)

---

## 28.3 The OIS-IBOR Basis: Discounting vs. Projection

### 28.3.1 What Is the OIS-IBOR Basis?

As developed in Chapter 20, the post-crisis derivatives world uses separate curves for different purposes:
- **OIS Curve**: Used for *discounting* collateralized cashflows
- **IBOR Projection Curves**: Used for *estimating* future floating rate fixings

The spread between these curves is the **OIS-IBOR basis** (or in the modern SOFR-dominated world, the spread between term rates and overnight rates).

In stress, term unsecured funding rates can rise far above overnight secured/collateralized rates. This gap was one motivation for the multi-curve framework and for using an overnight collateral curve for discounting in many derivatives contexts.

This basis is not an anomaly to be “arbitraged away” for free: it reflects genuine economic differences $credit horizon, liquidity, and balance-sheet constraints$ between overnight and term funding.

### 28.3.2 Why the Basis Exists

Chapter 20 covered the economic drivers of tenor basis. For the OIS-IBOR basis specifically:

**Credit Horizon**: A bank borrowing for 3 months faces more credit risk than rolling overnight funding. The term rate must compensate for this additional risk.

**Liquidity Preference**: Banks structurally prefer to match their funding duration to their assets. The desire for term funding creates a premium.

**Counterparty Concentration**: Term lending concentrates credit exposure to a single counterparty; overnight lending allows daily reassessment.

> **Analogy: Celsius vs. Fahrenheit**
>
> Basis trades are like measuring the temperature of money using two different thermometers.
>
> - **Thermometer A (OIS)**: Measures the "Risk-Free" temperature. (Is money expensive because the Fed hiked rates?)
> - **Thermometer B $LIBOR/IBOR$**: Measures the "Bank Risk" temperature. (Is money expensive because banks are scared to lend to each other?)
>
> In normal times, they move together $0°C = 32°F$. But in a crisis, Thermometer B spikes while Thermometer A might stay low.
>
> **The Basis Trade**: You are betting on the *difference* between the readings. If you think the gap is too wide, you sell Thermometer B and buy Thermometer A.
>
> **Visual: The Basis Scale**
>
> - **Left Side (OIS)**: Heavy, stable, clean. Represents the "Risk-Free" anchor.
> - **Right Side (IBOR)**: Lighter, volatile, dirty. Represents "Bank Risk."
>
> A Basis Swap balances the scale. If the Right Side gets heavier (Credit Risk rises), you must add more weight (Spread) to the Left Side to keep it balanced. The "Spread" is the fear premium.

### 28.3.3 Trading the OIS-IBOR Basis

**Instruments**: OIS swaps, Fed funds/SOFR basis swaps

**Trade Structure**: Enter a basis swap receiving Term SOFR (or historically LIBOR) and paying OIS + spread

**What You're Long**:
- Bank credit risk / term funding premium
- Liquidity premium in term rates
- Widening of the basis during stress

**What You're Short**:
- The "risk-free" overnight rate
- The "flight to safety" trade

### 28.3.4 Risk Decomposition in the Multi-Curve Framework

In a multi-curve world, it is helpful to separate three conceptually different “rate risks”:

| Risk Type | What Drives It | Hedge Instrument |
|-----------|---------------|------------------|
| **Rate Level** | Overall rates move up/down | Standard swaps, futures |
| **Discounting** | OIS curve moves | OIS swaps |
| **Basis** | Spread between curves moves | Basis swaps |

The framework allows traders to isolate specific risk exposures while hedging unwanted ones.

**Expand (what is held fixed):** These “three risks” only become operationally meaningful when you state a bump rule.

- A **level** move usually means “shift the relevant curve set together” (e.g., bump both discount and projection curves in a consistent way).
- A **discounting** move means “bump the discount curve only” (hold the projection curve fixed).
- A **basis** move means “bump a *basis quote* $or basis-spread parameter$ and rebuild the affected projection curve(s), holding the discount curve fixed.”

Two trades can both be “DV01-neutral” to a level move and still have very different discount/basis exposures because they are sensitive to *different objects*.

**Check (toy scaling):** For a floating–floating basis swap where you pay “OIS + $e$” on notional $N$, the PV impact of a 1 bp widening in the quoted basis is of order $N \times \text{basis annuity} \times 10^{-4}$. If $N=\$100\text{mm}$ and the discounted annuity is about $4.2$ years, then a 1 bp move is roughly $\$100\text{mm}\times 4.2\times 10^{-4}\approx \$42{,}000$ (sign depends on whether you pay or receive $e$).

### 28.3.5 Why OIS-IBOR Basis Moves

**The basis widens when:**
- **Credit stress**: Bank credit concerns push up term rates relative to OIS
- **Liquidity hoarding**: Banks prefer to hold cash rather than lend at term tenors

**The basis narrows when:**
- **Risk-on sentiment**: Credit concerns abate
- **Liquidity improves**: Term funding premia fall relative to overnight

### 28.3.6 Practical Considerations for Basis Trading

**Correlation with Other Positions**: OIS-IBOR basis is often *correlated* with other credit spreads. A position that is long basis may perform poorly precisely when your other spread positions (corporate bonds, CDS protection sales) also suffer. This is wrong-way risk in disguise.

**Mark-to-Market Volatility**: Basis positions can show large interim P&L swings. Even if your long-term view is correct, position limits or margin calls may force liquidation at the worst time.

### 28.3.7 Mathematical Foundations of Multi-Curve Pricing

This section provides the rigorous mathematical framework underlying multi-curve basis trades.

#### OIS Payoff and Discounting

The OIS floating payment at maturity $T$ is derived from compounding daily overnight rates. The compounded growth factor is:

$$\boxed{R_{01}(t,T) = \prod_{i=1}^{M}\bigl(1 + f_{i-1}\,\tau_i\bigr),}$$

where $f_i$ are daily overnight rates and $\tau_i$ are day-count fractions. The floating payment is $\displaystyle\frac{R_{01}(0,T) - 1}{\tau(0,T)}$ (scaled by accrual).

**Unit check:** $R_{01}$ is dimensionless (a growth factor). $(R_{01} - 1)/\tau$ is an annualized rate ($1/\text{year}$).

When OIS rates are used for discounting, you must build a zero curve from OIS rates analogous to building a swap zero curve from swap rates.

#### Vanilla Swap PV in Multi-Curve Framework

Generic swap PV is a sum of discounted net cashflows:

$$V_{\text{swap}}(t) = \sum_n \tau_n\,P(t,T_{n+1})\,(L_n(t) - k_n),$$

where $P(t,T)$ is the discount factor and $L_n$ comes from the relevant projection curve.

Under multi-curve, a clean "rates-only" PV (per notional) is:

$$PV_{\text{swap}}(0) = \sum_{i=1}^{n} \tau_i\,P_d(0,T_i)\bigl(L_{\text{IBOR}}(0,T_{i-1},T_i) - K\bigr).$$

Discount vs projection appears explicitly:
- $P_d(0,T_i)$ — discount curve
- $L_{\text{IBOR}}$ — projection curve

#### Multi-Curve Par Rate Formula

Setting $PV_{\text{swap}}(0) = 0$ and solving:

$$\boxed{K_{\text{par}}^{\text{(multi)}} = \frac{\sum_{i=1}^{n} \tau_i P_d(0,T_i)\,L_{\text{IBOR}}(0,T_{i-1},T_i)}{\sum_{i=1}^{n} \tau_i P_d(0,T_i)}.}$$

This is the "discounted average" of projected forwards.

**Unit check:**
- Numerator: (year) × (dimensionless) × (1/year) summed ⇒ dimensionless
- Denominator: (year) × (dimensionless) summed ⇒ years
- Ratio: 1/year (a rate) ✅

#### Basis Swap Par Condition

A floating–floating basis swap exchanges floating payments linked to two indices $L_1$ and $L_2$, typically plus a quoted spread $e$ on one leg. At par, PV of legs must match:

$$\sum_i L_2(0,t_i^2,t_{i+1}^2)\,\tau_i^2\,P(t_{i+1}^2) = \sum_i \bigl(L_1(0,t_i^1,t_{i+1}^1) + e_{1,2}(T)\bigr)\,\tau_i^1\,P(t_{i+1}^1),$$

with $e_{1,2}(T)$ quoted on the $L_1$ leg and possibly positive or negative.

For the common case of same payment schedule, solving for the par basis spread:

$$\boxed{e_{\text{par}} = \frac{\sum_i \tau_i P_d(0,T_i)(L_i^2 - L_i^1)}{\sum_i \tau_i P_d(0,T_i)}.}$$

#### Basis DV01 (Sensitivity to Quoted Basis)

For a position where $e$ is paid (i.e., PV decreases when $e$ increases):

$$\frac{\partial PV}{\partial e} = -N \sum_i \tau_i P_d(0,T_i), \quad\text{so}\quad \Delta PV \approx -N\Bigl(\sum_i \tau_i P_d(0,T_i)\Bigr)\Delta e.$$

If $\Delta e = +1\text{ bp} = 10^{-4}$, then "basis PV01" (in currency units) is:

$$\boxed{\text{Basis PV01} = -N\Bigl(\sum_i \tau_i P_d(0,T_i)\Bigr) \cdot 10^{-4}.}$$

**Check (units and sign):** The sum $\sum_i \tau_i P_d(0,T_i)$ has units of discounted years. Multiplying by $N$ gives discounted dollars, and multiplying by $10^{-4}$ turns a per-1 move into a per-1bp move. For example, if $N=\$100\text{mm}$ and $\sum_i \tau_i P_d\approx 4.2$, then paying the basis ($e$ up makes PV down) gives $\text{Basis PV01}\approx -\$100\text{mm}\times 4.2\times 10^{-4}=-\$42{,}000$ per bp, while receiving the basis flips the sign.

#### Discount PV01 vs. Projection PV01

**Discount PV01** (finite-difference definition):

$$\text{PV01}_{\text{discount}} \equiv PV(P_{d,+1\text{bp}}, \{L_k\}) - PV(P_d, \{L_k\}),$$

where only the discount curve is bumped.

**Projection PV01** (finite-difference definition):

$$\text{PV01}_{\text{projection}}^{(k)} \equiv PV(P_d, L_{k,+1\text{bp}}) - PV(P_d, L_k),$$

holding discount curve fixed.

**Practical Note:** In multi-curve PV formulas, every cashflow is multiplied by $P_d(0,T)$. Even if projection forwards are unchanged, changing $P_d$ changes PV. For near-par swaps, projection PV01 often dominates discount PV01 because forward bumps change coupon amounts directly while discount bumps rescale both legs and can partially cancel.

---

## 28.4 Treasury Futures Basis: The Cash-Futures Trade

The Treasury futures basis trade is often called simply "the basis trade" on rates desks. It is the strategy that keeps futures priced correctly relative to cash bonds—and a strategy that can experience severe stress when funding and margin mechanics interact badly. This section develops the mechanics in detail.

### 28.4.1 Gross Basis and Net Basis

Let $P^i(t)$ be the spot price of bond $i$ at time $t$, let $P^i_{fwd}(t)$ be its forward price to the last delivery date, let $F(t)$ be the futures price, and let $cf^i$ be the conversion factor. Then:

$$\boxed{GB^i(t) = P^i(t) - cf^i \times F(t)}$$

$$\boxed{NB^i(t) = P^i_{fwd}(t) - cf^i \times F(t) = GB^i(t) - \text{carry}^i(t)}$$

The term *net* basis is “gross basis net of carry.”

At delivery, the forward price equals the spot price (carry equals zero), so gross basis equals net basis. Furthermore, both measures equal the cost of delivery.

**Check (units and a toy decomposition):** $GB$, $NB$, and “carry” are all **price units** (points per 100 of face), not dollars. A simple decomposition is:
- if $GB=+0.50$ points and carry-to-delivery is $+0.30$ points (coupon income exceeds financing), then $NB=GB-\text{carry}=+0.20$ points;
- for $G=\$100\text{mm}$ face, $0.20$ points corresponds to $\$100\text{mm}\times 0.20/100=\$200{,}000$ of net-basis PV (before mark-to-market and margin timing).

> **Pitfall — Gross vs. net basis (carry double-counting):** Gross basis is a spot–futures gap; net basis adjusts gross basis for carry/financing to delivery.
> **Why it matters:** If you forecast/mark P&L using net basis changes, and then separately add “carry”, you can overstate expected returns and mis-size leverage.
> **Quick check:** If you compute P&L as $G^i\,[NB^i(t')-NB^i(t)]$ (or its short-basis analog), stop—do **not** add coupon minus repo carry again unless you are using *gross* basis consistently.

### 28.4.2 The Long Basis Trade Construction

A long basis trade in bond $i$ can be implemented as:
- Buy $G^i$ face amount of deliverable bond $i$.
- Finance it in repo to the last delivery date.
- Sell $cf^i \times G^i / 100{,}000$ futures contracts.

Economically, “buy bond + repo financing” is a bond **forward** position to the delivery date, so the package is long the bond forward and short the futures (in conversion-factor-adjusted size).

In a frictionless description this looks “zero-cash” (repo finances the bond; a futures position is entered at zero cost), but in practice you must fund **repo haircuts** and **futures margin**, so the trade consumes capital and can be forced out by funding/margin shocks.

**Long Basis Trade:**
- Buy bond, finance via repo, sell futures
- Profit if net basis *widens* (you own a forward that becomes more valuable relative to futures)

**Short Basis Trade:**
- Sell bond, invest proceeds, buy futures
- Profit if net basis *narrows*

### 28.4.3 P&L Formula

The P&L from a long basis trade is remarkably simple:

$$\boxed{\text{P\&L} = G^i \times [NB^i(t') - NB^i(t)]}$$

This formula encapsulates everything: you profit if the net basis widens, lose if it narrows.

**Example Title**: Net-basis P&L on a short-basis position (ticks → dollars)

**Context**
- You are short net basis in a deliverable bond vs futures: you expect convergence (net basis to fall) and you are trying to be close to rate-neutral.
- The goal is to see how small tick moves translate into large dollar P&L on big notionals.

**Timeline (Make Dates Concrete)**
- Trade date: 2000-02-28
- Horizon / mark date: 2000-05-19
- Last delivery date: 2000-06-30

**Inputs**
- Face amount: $G = \$100{,}000{,}000$.
- Net basis at entry: $NB_{\text{entry}} = 7.45$ ticks.
- Scenario A (convergence): $NB = 3.00$ ticks.
- Scenario B (adverse widening): $NB = 22.00$ ticks.
- Quote convention: 1 tick = $1/32$ of a price point; price points are per 100 face.

**Outputs (What You Produce)**
- P&L at horizon for a **short-basis** position:
  - Scenario A: +\$139{,}063
  - Scenario B: -\$454{,}688
- Scaling: for $G=\$100\text{mm}$ face, 1 tick $\approx \$31{,}250$.

**Step-by-step**
1. Tick change: $\Delta NB_{\text{ticks}} = NB_{\text{exit}} - NB_{\text{entry}}$.
2. Convert ticks to points per 100: $\Delta P = \Delta NB_{\text{ticks}}/32$.
3. Convert to dollars: $P\&L_{\text{long}} = G \cdot (\Delta P/100)$, so $P\&L_{\text{short}} = -P\&L_{\text{long}}$.
4. Scenario A: $\Delta NB = 3.00-7.45=-4.45$ ticks ⇒ $P\&L_{\text{short}}=+\$100\text{mm}\cdot(4.45/32)/100=+\$139{,}063$.
5. Scenario B: $\Delta NB = 22.00-7.45=14.55$ ticks ⇒ $P\&L_{\text{short}}=-\$100\text{mm}\cdot(14.55/32)/100=-\$454{,}688$.

**Cashflows (table)**
| Date | Cashflow | Explanation |
|---|---|---|
| 2000-05-19 | MTM P&L | From change in net basis; ignores funding cost and the *timing* of margin/haircut cashflows |

**P&L / Risk Interpretation**
- “Rate-neutral” is not the same as “path-safe”: net basis near CTD behaves like an option, so it can widen in either a rally or a selloff.
- Margin and repo haircuts can force liquidation before delivery convergence.

**Sanity Checks**
- Units: ticks → points per 100 → dollars.
- Sign: short basis profits when net basis falls.

### 28.4.4 What the Net Basis Represents: CTD Optionality

The net basis captures the value of the delivery options embedded in the futures contract. Under proper “tailing” conventions, the net basis of a particular bond can be interpreted as the value (with respect to that bond) of the short’s **quality option** at delivery: the cost of being committed to deliver bond $i$ rather than being free to deliver the CTD.

The short futures holder has embedded options:

**1. Quality Option**: The short can deliver any eligible bond, choosing the cheapest-to-deliver (CTD)

**2. Timing Option**: The short can choose when during the delivery month to deliver

**3. End-of-Month Option**: After the last trading day, the short can still deliver based on the final settlement price while bond prices may have moved

Key intuition: if a bond’s net basis is near zero, then the quality option “with respect to that bond” is nearly worthless—selling that bond forward is close to selling the futures contract (because the bond is close to CTD under current conditions).

**Expand (two-path story):** A Treasury future is economically like “sell a forward on the *cheapest* deliverable bond.” The short is allowed to pick which bond to deliver, so the futures short effectively owns an option to choose the cheapest invoice cost. That choice makes the futures price behave like a **minimum** over deliverable forward prices (after conversion-factor adjustment). Your bond-specific net basis $NB^i$ is then “how much worse it would be” to be forced to deliver bond $i$ instead of the eventual CTD.

**Check (limiting cases):**
- If one bond is *clearly* CTD under a wide range of yield scenarios, the quality option is small and its net basis tends to be close to the CTD-implied benchmark.
- If two bonds are near-indifferent (both plausible CTD), the quality option becomes valuable and net basis becomes more convex (“short volatility” for a short-basis trade), because small rate moves can flip the identity of CTD.

### 28.4.5 Net Basis Behavior: Like a Straddle on Rates

One of the most useful “feel” results is that a bond close to CTD can have net basis that behaves *like a straddle on yields*: it can increase when yields fall and also when yields rise, because either move can push the bond away from CTD and increase the value of the delivery optionality.

For bonds away from CTD, the behavior is directional:
- **Short-duration bonds**: Net basis behaves like a call on rates (put on prices)—rises when rates rise
- **Long-duration bonds**: Net basis behaves like a put on rates (call on prices)—rises when rates fall

This option-like behavior explains why basis trades have significant tail risk: a large rate move in either direction can cause substantial losses for a short basis position in a near-CTD bond.

> **Desk Reality:** Near-CTD short-basis positions are implicitly short “CTD-switch” optionality (short volatility).
> **Common break:** A parallel rally or selloff pushes your bond away from CTD and the net basis widens, generating losses even if you expect delivery convergence.
> **What to check:** Stress net basis under $\pm 50$ bp parallel shifts and plausible CTD switches; size so that interim widening + margin/haircuts are survivable (options are sometimes used to cap the tail).

### 28.4.6 CTD Dynamics: When the Cheapest Bond Changes

As discussed in Chapter 23, the CTD bond depends on yield levels and curve shape:

**High yields (above notional coupon):** Short-duration bonds become CTD
**Low yields (below notional coupon):** Long-duration bonds become CTD
**Steep curve:** Longer-duration bonds become relatively cheaper

When yields move significantly, the CTD can switch to a different bond. For a short basis position, a CTD switch means your bond has moved *further* from CTD, widening your net basis and generating losses.

> **Worked Example: CTD Switch P&L Impact**
>
> TYH2 basket on November 26, 2001 (from Tuckman Table 20.5):
>
> | Bond | Maturity | Net Basis (ticks) |
> |------|----------|-------------------|
> | 6.00% | 08/15/09 | 13.6 |
> | 6.50% | 02/15/10 | 13.8 |
> | 5.50% | 05/15/09 | 16.4 |
> | 4.75% | 11/15/08 | 17.6 |
>
> The 6s of August '09 and 6.50s of February '10 are essentially jointly CTD (net bases within 0.2 ticks).
>
> **Scenario:** Yields fall 50bp in parallel
> - Short-duration bonds (4.75s, 5.50s) move toward CTD
> - The 6.50s move away from CTD
> - Suppose the net basis of the 6.50s widens from 13.8 to 25.0 ticks
>
> A \\\$50mm short basis position in the 6.50s would lose approximately:
> - $50mm × (25 - 13.8)/32 / 100 = $175,000

---

## 28.5 Curve Relative Value: The Spread of Spreads

### 28.5.1 What Is a Spread of Spreads Trade?

Often the absolute level of a spread is uncertain, but the *relative* relationship between two spreads appears mispriced. This leads to "spread of spreads" trades—positions designed to profit from the convergence of two spreads regardless of their absolute levels.
In trader language, you are expressing a view on *relative richness/cheapness* (spread A vs spread B), not on the absolute level of rates.

### 28.5.2 Example: Trading TED Spread Differentials

**Setup**: Two bonds have different TED spreads (spreads to Eurodollar/SOFR futures rates):
- Bond A: TED spread of 15.6 bp
- Bond B: TED spread of 20.5 bp

**Trade**: Buy Bond A, sell Bond B, hedge each with the appropriate futures strip

**What You're Long**:
- Bond A richening relative to Bond B
- The "spread of spreads" narrowing

**What You're Short**:
- Bond B's cheapness relative to Bond A

**Critical Hedge Construction**: A spread-of-spreads trade should be (approximately) DV01-neutral (or key-rate-neutral) so that a parallel shift in rates does not dominate the P&L.

The trade must be constructed so that:
1. Each bond is hedged against its own rate risk using futures
2. The combined DV01 of both positions is zero
3. The net futures position is approximately zero

Only then does the trade generate P&L solely from the spread of spreads.

### 28.5.3 The Five-Year OTR/Old Five-Year Trade

A classic spread-of-spreads example is trading the spread between on-the-run and off-the-run Treasuries via asset swaps.

**Setup on January 17, 2001:**
- OTR 5Y (5.75% Nov 2005): Yield 4.852%, Spread to swaps -91.3 bp
- Old 5Y (6.75% May 2005): Yield 4.971%, Spread to swaps -75.4 bp
- Spread of spreads: -15.9 bp (OTR was 15.9 bp richer to swaps)

**The Question**: Is a spread of spreads of 15.9 basis points justified? Both bonds are Treasuries; the swap curve over six months reflects nearly identical rolling bank credit. The experienced trader argued that on a forward basis to May 8, 2001, accounting for financing and liquidity effects, the spread of spreads should be only about 3 basis points (the typical old-to-double-old premium).

**The Trade**: Sell the OTR asset swap, buy the old 5Y asset swap, sized so DV01s match.

**Result (computed below)**: The forward spread of spreads narrows from 7.6 bp to 2.9 bp, producing a profit of about $192k for a $100mm position with forward DV01 0.04086 per bp.

> **Worked Example: Spread of Spreads P&L**
>
> **Initial Position (Jan 17, 2001):**
> - Short \\\$100mm OTR 5Y asset swap at -91.3 bp spread
> - Long offsetting DV01 in old 5Y asset swap at -75.4 bp spread
> - Forward spread of spreads = 7.6 bp
>
> **Terminal Position (May 8, 2001):**
> - OTR 5Y spread = -80.1 bp
> - Old 5Y spread = -77.2 bp
> - Spread of spreads = 2.9 bp
>
> **P&L Calculation:**
> - Change in spread of spreads = 7.6 - 2.9 = 4.7 bp
> - Forward DV01 = 0.04086 per bp
> - P&L ≈ $100mm × 0.04086 × 4.7 = **$192,042**
>
> The trade generated P&L purely from the spread-of-spreads convergence.

### 28.5.4 Swap Curve Relative Value

The same logic applies to the swap curve itself:

**Example: Trading the 5Y vs. 10Y Swap Spread Differential**

If the 5Y swap spread is 50bp and the 10Y is 80bp, a trader might believe the 30bp "spread of spreads" is too wide—perhaps because curve steepening has mechanically widened 10Y spreads without fundamental justification.

**Trade**:
- Receive fixed on 5Y swap, short 5Y Treasury
- Pay fixed on 10Y swap, long 10Y Treasury

Sized so that:
- 5Y position and 10Y position have offsetting DV01
- The trade profits if the spread differential narrows, regardless of the absolute level of swap spreads

---

## 28.6 What Can Go Wrong: Case Studies

This section examines three historical episodes where basis trades failed catastrophically. Understanding these failures is essential for appreciating why basis trades are *not* arbitrage—they are convergence trades with significant tail risk.

### 28.6.1 Case Study 1: Long-Term Capital Management, 1998

LTCM is a canonical failure mode for convergence/RV trades. The investment strategy was known as **convergence arbitrage**: buy a less-liquid instrument and short a corresponding more-liquid instrument, expecting their prices to converge.

In the summer of 1998, a Russian debt default triggered a flight to quality. LTCM tended to be long illiquid instruments and short the corresponding liquid instruments (e.g., long off-the-run bonds and short on-the-run bonds). The spreads widened sharply. Because LTCM was highly leveraged, the losses generated margin calls that were difficult to meet.

Trading takeaway: run scenario analysis and stress testing for “worst of all worlds” outcomes, not just expected convergence.

### 28.6.2 Case Study 2: Funding Shock (Repo and Term Funding)

Even outside marquee crisis windows, funding liquidity can tighten abruptly. For a basis trade that is long a cash bond financed in repo, higher repo rates and/or worse financing terms can:
- Reduce carry (coupon minus financing cost).
- Force deleveraging if the financing horizon shortens or haircuts rise.

**Toy sizing example (illustrative, not historical):** financing \$1 billion overnight, moving repo from 2% to 10% changes daily interest cost from
- $\$1\text{bn} \times 2\% \times (1/360) = \$55{,}556$ per day to
- $\$1\text{bn} \times 10\% \times (1/360) = \$277{,}778$ per day,
an extra \$222{,}222 per day. On a trade targeting a few ticks/bp of convergence, that can dominate the economics.

Lesson: term repo reduces funding cliff risk but usually costs more; sizing needs to reflect the worst plausible funding path, not just the expected convergence.

### 28.6.3 Case Study 3: Margin and Liquidity Feedback Loop (Cash–Futures)

Cash–futures basis trades combine two very different settlement mechanics.

Futures positions are settled daily: at the end of each day the position is marked to market and gains/losses are settled through variation margin (pay variation margin when you lose; receive it when you gain). In significant price volatility, intraday variation margin can also be required; if margin requirements are not met, positions can be closed out.

Cash bonds are typically funded (e.g., via repo). They are marked to market, but the gain is not automatically converted into cash on the futures settlement timeline. Turning the bond gain into usable cash depends on repo terms or on selling the bond, and haircuts can rise.

This creates a common failure mode: the position can be “economically hedged” but fail a liquidity test. You can owe cash on the futures leg today while the bond gain is not immediately usable (or becomes expensive to monetize) on the same timeline.

> **Desk Reality:** A basis book can be PV-hedged and still blow up on cashflow timing.
> **Common break:** Futures variation margin outflows arrive faster than bond P&L can be monetized, forcing deleveraging into a widening basis.
> **What to check:** run a liquidity stress that combines (i) adverse basis move, (ii) higher haircuts/margins, and (iii) reduced funding horizon.

---

## 28.7 Hidden Risks and Practical Execution

### 28.7.1 Funding Risk

Almost every basis trade requires financing. Changes in funding costs directly affect P&L.

In an asset-swap-like package, the net “carry” is roughly: floating index + spread − financing rate. If financing spikes, carry shrinks or turns negative.

If you enter a basis trade earning a spread over financing, and financing rates spike, your periodic income shrinks or turns negative. This is particularly dangerous for leveraged positions where the spread earned is small relative to the total financing cost.

**Mitigants:**
- Use term repo (lock in financing for the trade duration)
- Size positions to survive a funding spike
- Monitor repo specials for your specific bonds

### 28.7.2 Roll and Carry

Basis trades age. The passage of time affects P&L through multiple channels:

**Curve roll**: If the yield curve is upward sloping, instruments may "roll down" to lower yields as time passes. A long bond position benefits from this; a swap position may or may not, depending on the curve shape.

**Carry**: Coupon income minus financing cost determines whether a position earns or bleeds money while you wait for your spread thesis to materialize.

**Convergence**: Futures basis must converge to the cost of delivery at expiration. A long basis position that is held to delivery will earn the initial net basis (minus the delivery option value at expiration).

### 28.7.3 Liquidity Risk: The Crowded Trade Problem

This is the danger of "crowded trades"—when many participants have the same basis position, adverse moves trigger forced unwinds. Those liquidations can further widen the basis, creating a feedback loop. A trade can be “fundamentally” right and still be forced out on the path.

### 28.7.4 Convexity Mismatches

For large rate moves, the different convexity profiles of the instruments in a basis trade create P&L that was not part of the original thesis.

A major example is mortgage convexity hedging. When rates fall, MBS duration shortens (negative convexity), and hedgers receive fixed in swaps to compensate. This concentrated receiving pressure can narrow swap spreads. When rates rise, the reverse occurs—hedgers pay fixed, widening spreads.

The implication: swap spreads exhibit "directionality." They tend to widen in selloffs and narrow in rallies, not because of credit fundamentals, but because of the convexity hedging needs of the MBS market. A trader who is short swap spreads (receiving fixed, short Treasuries) is implicitly short the volatility of the mortgage market.

### 28.7.5 Margin and Capital Mechanics

Understanding the difference between futures and cash positions is essential:

**Futures:**
- Daily mark-to-market and margin settlement
- Margin calls must be met in cash, typically same-day
- Variation margin moves daily; initial margin ties up capital

**Cash Bonds (financed via repo):**
- No daily margin (unless using cleared repo)
- Haircuts tie up capital: a 2% haircut means \$2mm capital per \$100mm financed
- Repo can be overnight (funding cliff risk) or term (more expensive)

**The Asymmetry:**

A basis trade that is long cash bonds and short futures faces asymmetric margin dynamics:
- When rates fall: bonds gain, futures lose → futures margin call (must pay cash)
- When rates rise: bonds lose, futures gain → futures margin credit (receive cash), but bond may be underwater

The daily margining of futures creates cash flow volatility even when the net P&L is stable. This can create liquidity crises in volatile markets.

### 28.7.6 Sizing the Trade

Basis trades should be sized based on three factors:

**Maximum drawdown tolerance**: How much interim widening can you sustain without being forced to exit? Even trades that eventually converge can experience large drawdowns on the path.

**Bid-offer costs**: Basis trades often have multiple legs; each has execution cost. A trade earning 5bp that costs 2bp to enter and 2bp to exit has thin economics.

**Margin and capital**: Futures require margin; repos require haircuts; swaps require capital under regulatory frameworks. Size constraints often bind before risk limits.

### 28.7.7 Entry and Exit

**Entry**: Look for both technical dislocations (forced selling, balance-sheet constraints) and fundamental mispricings. The best basis trades combine a sound economic thesis with a catalyst that will cause convergence.

**Exit**: Define the exit criteria before entry:
- Target profit level
- Stop-loss level
- Time horizon (some basis trades "work" only if held to maturity or to delivery)

### 28.7.8 Monitoring Residuals

Throughout the life of the trade:
- Rebalance DV01 as needed (hedge ratios drift)
- Monitor funding costs
- Track positioning indicators (is everyone doing this trade?)
- Watch for regime changes that alter the thesis

### 28.7.9 Practical Execution Checklist

When you write up (or review) any rates basis/RV trade, explicitly list:

| Exposure | Questions to Answer |
|----------|---------------------|
| **Discount curve exposure** | Are you long/short OIS discount factors $P_d$? Do you report $\text{PV01}_{\text{discount}}$ separately? |
| **Projection curve exposure** | Which forward curve(s) drive your coupons (IBOR 3M, IBOR 6M, OIS, etc.)? Do you report $\text{PV01}_{\text{projection}}^{(k)}$ per index? |
| **Quoted basis exposure** | Which basis quote(s) enter PV (e.g., $e_{1,2}(T)$)? What is your basis PV01 (per \$1mm)? |
| **Benchmark choice exposure (swap spreads)** | Are you measuring swap spread vs on-the-run Treasury yields, or fitted/interpolated Treasury curve yields? Tuckman emphasizes this choice can distort the signal due to liquidity and special financing in on-the-run issues. |
| **Funding/repo exposure (cash legs)** | If you hold/short a cash bond, what repo rate is assumed? What is carry (interest income − financing cost) and how sensitive is it to repo moves? |
| **Convexity and curve-shape residuals** | Are you truly DV01-neutral (parallel) but exposed to twists/curvature? Are there convexity mismatches? |

---

## 28.8 Extended Worked Examples

This section provides twelve comprehensive numerical examples covering multi-curve valuation, basis swap pricing, risk decomposition, DV01 hedging, and curve relative value trades. All examples use toy numbers for pedagogy and state conventions explicitly.

---

### Example A — OIS vs IBOR Swap Valuation Difference (Same Swap, Two Frameworks)

**Goal:** Price the same 3Y pay-fixed swap under:
1. legacy single-curve (IBOR projects and discounts),
2. multi-curve (OIS discounts, IBOR projects),

and quantify PV and par-rate differences.

#### A.1 Conventions

- Notional ($N = \$1{,}000{,}000$).
- Annual payments at $T_1 = 1$, $T_2 = 2$, $T_3 = 3$.
- Accruals ($\tau_i = 1$) (toy).
- Swap direction: pay fixed $K$, receive floating IBOR.

**Legacy single-curve inputs (IBOR curve):** discount factors
$$P_L(0,1) = 0.9650,\quad P_L(0,2) = 0.9250,\quad P_L(0,3) = 0.8850.$$

**Multi-curve inputs:**
- Discount factors (OIS):
$$P_d(0,1) = 0.9700,\quad P_d(0,2) = 0.9400,\quad P_d(0,3) = 0.9100.$$

- Projection forwards (IBOR) taken from the legacy curve via simple no-arb identities:
$$\begin{aligned}
L(0,0,1) &= \frac{1}{P_L(0,1)} - 1 = \frac{1}{0.9650} - 1 = 0.036269,\\
L(0,1,2) &= \frac{P_L(0,1)}{P_L(0,2)} - 1 = \frac{0.9650}{0.9250} - 1 = 0.043243,\\
L(0,2,3) &= \frac{P_L(0,2)}{P_L(0,3)} - 1 = \frac{0.9250}{0.8850} - 1 = 0.045197.
\end{aligned}$$

(Toy: forwards remain unchanged between frameworks.)

#### A.2 Step 1 — Compute Legacy Single-Curve Par Rate (K_{\text{par}}^{\text{(single)}})

Use the par swap-rate concept (floating leg at par implies fixed leg at par).

**Annuity:**
$$A_L = \sum_{i=1}^{3} \tau_i P_L(0,T_i) = 0.9650 + 0.9250 + 0.8850 = 2.7750.$$

**Float PV (single-curve):** $1 - P_L(0,3) = 1 - 0.8850 = 0.1150$.

**Par fixed rate:** $K_{\text{par}}^{\text{(single)}} = \frac{0.1150}{2.7750} = 0.04144 \approx 4.144\%$.

Define the "same swap" as the contractual fixed rate:
$K \equiv 4.144\%$.

#### A.3 Step 2 — Price the Same Swap Under Legacy Single-Curve

Because $K$ is the single-curve par rate, $PV_{\text{single}} = 0$ (by construction).

#### A.4 Step 3 — Price Under Multi-Curve (OIS Discounting, IBOR Projection)

PV per unit notional (toy linear PV):
$$PV_{\text{multi}}(0) = \sum_{i=1}^{3} \tau_i P_d(0,T_i)\bigl(L(0,T_{i-1},T_i) - K\bigr).$$

Compute each term:
- $T_1$: $L - K = 0.036269 - 0.04144 = -0.005171$;
  $0.9700 \times (-0.005171) = -0.005016$.

- $T_2$: $L - K = 0.043243 - 0.04144 = +0.001803$;
  $0.9400 \times 0.001803 = 0.001694$.

- $T_3$: $L - K = 0.045197 - 0.04144 = +0.003757$;
  $0.9100 \times 0.003757 = 0.003418$.

**Sum:** $PV_{\text{multi}}/N \approx -0.005016 + 0.001694 + 0.003418 = 0.000095$.

So: $PV_{\text{multi}} \approx 0.000095 \times 1{,}000{,}000 = \$95$.

#### A.5 Step 4 — Compare Par Rates Under the Two Frameworks

**Multi-curve annuity:** $A_d = 0.9700 + 0.9400 + 0.9100 = 2.8200$.

**Multi-curve numerator:** $\sum P_d L = 0.9700(0.036269) + 0.9400(0.043243) + 0.9100(0.045197) = 0.116959$.

**Multi-curve par rate:** $K_{\text{par}}^{\text{(multi)}} = \frac{0.116959}{2.8200} = 0.041475 \approx 4.1475\%$.

#### A.6 Interpretation ("Discount vs Projection Basis")

- The same contractual swap can have different PV when discounting switches from a legacy IBOR curve to OIS $multi-curve$.
- The par rate shift $K_{\text{par}}^{\text{(multi)}} - K_{\text{par}}^{\text{(single)}} \approx 0.34$ bp is a clean way to quantify the discounting/projection separation in this toy setup.

---

### Example B — OIS–IBOR Basis Swap Par Condition and Par Basis Spread

**Goal:** Write the PV equation for an OIS–IBOR basis swap and solve for the par basis spread.

#### B.1 Conventions

- Notional $N = \$1{,}000{,}000$.
- Maturity $T = 2$ years.
- Semiannual payments: $T = \{0.5, 1.0, 1.5, 2.0\}$, $\tau = 0.5$.

**Discounting curve:** OIS discount factors (toy):
$$P_d(0,0.5) = 0.9850,\; P_d(0,1) = 0.9700,\; P_d(0,1.5) = 0.9550,\; P_d(0,2) = 0.9400.$$

**Projection curves:**
- IBOR-6M forwards (toy inputs):
$$L_{\text{IBOR}} = \{3.60\%, 3.80\%, 4.00\%, 4.10\%\} \text{ for the four periods}.$$

- OIS forwards (implied from discount factors; simple-comp, consistent with the curve):
$$L_{\text{OIS}}(0,T_{i-1},T_i) = \frac{1}{\tau}\Bigl(\frac{P_d(0,T_{i-1})}{P_d(0,T_i)} - 1\Bigr).$$

**Quoting convention (must be explicit):**

We define:
- $L_1 \equiv$ OIS leg,
- $L_2 \equiv$ IBOR leg,
- quoted spread $e$ is added to the OIS leg (so the OIS leg pays $L_{\text{OIS}} + e$).

#### B.2 Step 1 — Compute OIS Forwards from Discount Factors

Using $\tau = 0.5$:

- $0 \to 0.5$:
$$L_{\text{OIS}} = \frac{1}{0.5}\Bigl(\frac{1}{0.9850} - 1\Bigr) = 2(1.015228 - 1) = 0.030456 = 3.0456\%.$$

- $0.5 \to 1.0$:
$$L_{\text{OIS}} = 2\Bigl(\frac{0.9850}{0.9700} - 1\Bigr) = 2(1.015464 - 1) = 0.030928 = 3.0928\%.$$

- $1.0 \to 1.5$:
$$L_{\text{OIS}} = 2\Bigl(\frac{0.9700}{0.9550} - 1\Bigr) = 0.031416 = 3.1416\%.$$

- $1.5 \to 2.0$:
$$L_{\text{OIS}} = 2\Bigl(\frac{0.9550}{0.9400} - 1\Bigr) = 0.031914 = 3.1914\%.$$

#### B.3 Step 2 — Write Par Condition and Solve for $e$

From the par floating–floating basis swap condition, PV(IBOR leg) = PV(OIS+spread leg).

Using common schedule:
$$\sum_{i=1}^{4} \tau P_d(0,T_i)\,L_i^{\text{IBOR}} = \sum_{i=1}^{4} \tau P_d(0,T_i)\,(L_i^{\text{OIS}} + e).$$

Rearrange:
$e = \frac{\sum_{i=1}^{4} \tau P_d(0,T_i)(L_i^{\text{IBOR}} - L_i^{\text{OIS}})}{\sum_{i=1}^{4} \tau P_d(0,T_i)}$.

**Compute weights** $w_i = \tau P_d(0,T_i)$:
- $w_{0.5} = 0.5(0.9850) = 0.4925$
- $w_{1.0} = 0.5(0.9700) = 0.4850$
- $w_{1.5} = 0.5(0.9550) = 0.4775$
- $w_{2.0} = 0.5(0.9400) = 0.4700$

**Sum** $W = \sum w_i = 1.9250$.

**Compute differences** $\Delta_i = L_i^{\text{IBOR}} - L_i^{\text{OIS}}$:
- $0 \to 0.5$: $0.03600 - 0.030456 = 0.005544$
- $0.5 \to 1.0$: $0.03800 - 0.030928 = 0.007072$
- $1.0 \to 1.5$: $0.04000 - 0.031416 = 0.008584$
- $1.5 \to 2.0$: $0.04100 - 0.031914 = 0.009086$

**Compute numerator:**
$$\sum w_i \Delta_i = 0.4925(0.005544) + 0.4850(0.007072) + 0.4775(0.008584) + 0.4700(0.009086) \approx 0.014530.$$

**Therefore:**
$$e \approx \frac{0.014530}{1.9250} = 0.007548 \approx 0.7548\% = 75.5\text{ bp}.$$

#### B.4 Sanity Check

Because IBOR forwards are higher than OIS forwards in all periods, the spread $e$ added to the OIS leg must be positive to make the legs equal PV. ✅

---

### Example C — Discount PV01 vs Projection PV01 for a Swap (Finite-Difference)

**Goal:** Compute finite-difference sensitivities:
- $\text{PV01}_{\text{discount}}$: bump OIS discount curve $+1$ bp, hold projection fixed,
- $\text{PV01}_{\text{projection}}$: bump IBOR projection curve $+1$ bp, hold discount fixed.

#### C.1 Conventions

- Use a 3Y annual-pay swap, same dates as Example A.
- Notional $N = \$1{,}000{,}000$.
- Pay fixed $K = 4.00\%$, receive IBOR forwards $L = \{3.6269\%, 4.3243\%, 4.5197\%\}$.
- Discount curve = OIS discount factors $P_d(0,1) = 0.97$, $P_d(0,2) = 0.94$, $P_d(0,3) = 0.91$.
- Accruals $\tau = 1$.

**Bump rules (must be explicit):**
- **Discount-curve bump:** parallel zero-rate bump $+1$ bp approximated by
$P_{d,+}(0,T) = P_d(0,T)\,e^{-0.0001\,T}$.

- **Projection-curve bump:** add $+1$ bp to each forward:
$L_i^{+} = L_i + 0.0001$.

In both cases, hold the other curve fixed.

#### C.2 Step 1 — Base PV

PV per unit notional:
$$PV(0) = \sum_{i=1}^{3} P_d(0,T_i)\,(L_i - K).$$

Compute (L_i - K):
- $T_1$: $0.036269 - 0.040000 = -0.003731$
- $T_2$: $0.043243 - 0.040000 = +0.003243$
- $T_3$: $0.045197 - 0.040000 = +0.005197$

Multiply by discount factors:
- $0.9700(-0.003731) = -0.003619$
- $0.9400(0.003243) = 0.003048$
- $0.9100(0.005197) = 0.004729$

**Sum:** $PV/N \approx -0.003619 + 0.003048 + 0.004729 = 0.004159$.

So:
PV \approx 0.004159 \times 1{,}000{,}000 = \$4{,}159.

#### C.3 Step 2 — Discount PV01 (Bump Discount Only)

**Compute bumped discount factors:**
- $T = 1$: $0.97\,e^{-0.0001} \approx 0.97(0.9999) = 0.969903$
- $T = 2$: $0.94\,e^{-0.0002} \approx 0.939812$
- $T = 3$: $0.91\,e^{-0.0003} \approx 0.909727$

Recompute PV with same $L_i$ and $K$:
$$PV_{d,+}/N \approx 0.969903(-0.003731) + 0.939812(0.003243) + 0.909727(0.005197).$$

Numerically:
$$\approx -0.003619 + 0.003048 + 0.004728 = 0.004157.$$

So $PV_{d,+} \approx \$4{,}157$.

**Therefore:**
$$\boxed{\text{PV01}_{\text{discount}} = PV_{d,+} - PV \approx 4{,}157 - 4{,}159 = -\$2 \text{ per \$1mm}.}$$

#### C.4 Step 3 — Projection PV01 (Bump Projection Only)

Bump forwards by $+1$ bp: $L_i^{+} = L_i + 0.0001$.

PV change per unit notional:
$$\Delta PV / N = \sum_{i=1}^{3} P_d(0,T_i) \cdot 0.0001 = 0.0001(0.97 + 0.94 + 0.91) = 0.000282.$$

So:
$$\boxed{\text{PV01}_{\text{projection}} \approx 0.000282 \times 1{,}000{,}000 = \$282 \text{ per } \$1\text{mm}.}$$

#### C.5 Interpretation ("Where the Risk Lives")

In this decomposition, the swap's PV is far more sensitive to projection +\$282/bp than to discounting $\approx -\$2/bp$.

**Intuition:** bumping forwards changes the size of projected coupons, while bumping discount factors changes primarily the present-valuing of already-nearly-offsetting legs.

**Caution:** If you instead bump market quotes and rebuild curves (par-point approach), the allocation between "discount PV01" and "projection PV01" can shift (Example L).

---

### Example D — Basis Spread Sensitivity ("Basis PV01") for Example B

**Goal:** PV change per $+1$ bp change in quoted basis spread $e$, per \$1mm notional.

#### D.1 Conventions

- Use Example B basis swap (2Y semiannual).
- Spread $e$ is on the OIS leg; position is receive IBOR, pay (OIS + e).
- Notional (N = \$1{,}000{,}000).
- Discount factors and weights from Example B: ($W = \sum \tau P_d = 1.9250$).

#### D.2 Calculation

PV of spread leg (per unit basis rate) is:
$$PV_{\text{spread}} = -N \sum_i \tau P_d(0,T_i)\,e = -N\,W\,e.$$

So per $+1$ bp ($\Delta e = 0.0001$):
$$\Delta PV \approx -N\,W\,0.0001 = -(1{,}000{,}000)(1.9250)(0.0001) = -\$192.50.$$

#### D.3 Output

$$\boxed{\text{Basis PV01 (for this position): } -\$192.50 \text{ per +1 bp of } e \text{ per } \$1\text{mm notional}.}$$

---

### Example E — Replicating a Basis Position with Two Swaps (Explicit PV Match)

**Goal:** Show a basis exposure can be represented as a portfolio of an IBOR swap and an OIS swap (toy, explicit) and match PV.

#### E.1 Conventions

- Use the same 2Y semiannual schedule as Example B.
- Discount curve $P_d$ same as Example B.
- Projection forwards:
  - IBOR forwards: $\{0.036, 0.038, 0.040, 0.041\}$.
  - OIS forwards: from Example B $\{0.030456, 0.030928, 0.031416, 0.031914\}$.
- Notional $N = \$1{,}000{,}000$.
- Accrual $\tau = 0.5$.
- Choose a common fixed rate for replication: $K_{\text{rep}} = 3.50\%$ (arbitrary).
- Define weights $w_i = \tau P_d(0,T_i)$ (from Example B):
$w = \{0.4925, 0.4850, 0.4775, 0.4700\}$.

#### E.2 Step 1 — PV of the "IBOR Pay-Fixed Swap" at $K_{\text{rep}}$

PV per unit notional:
$PV_{\text{IBORswap}} = \sum_i w_i\bigl(L_i^{\text{IBOR}} - K_{\text{rep}}\bigr)$.

Differences ($L_{\text{IBOR}} - K_{\text{rep}}$):
- $0.036 - 0.035 = 0.001$
- $0.038 - 0.035 = 0.003$
- $0.040 - 0.035 = 0.005$
- $0.041 - 0.035 = 0.006$

Weighted sum:
$PV/N = 0.4925(0.001) + 0.4850(0.003) + 0.4775(0.005) + 0.4700(0.006) = 0.007155$.

So $PV_{\text{IBORswap}} = \$7{,}155$.

#### E.3 Step 2 — PV of the "OIS Pay-Fixed Swap" at $K_{\text{rep}}$

$PV_{\text{OISswap}} = \sum_i w_i\bigl(L_i^{\text{OIS}} - K_{\text{rep}}\bigr)$.

Differences ($L_{\text{OIS}} - 0.035$):
- $0.030456 - 0.035 = -0.004544$
- $0.030928 - 0.035 = -0.004072$
- $0.031416 - 0.035 = -0.003584$
- $0.031914 - 0.035 = -0.003086$

Weighted sum:
$PV/N \approx -0.4925(0.004544) - 0.4850(0.004072) - 0.4775(0.003584) - 0.4700(0.003086) = -0.007375$.

So $PV_{\text{OISswap}} \approx -\$7{,}375$.

#### E.4 Step 3 — Replication Portfolio

Take:
- **Long** the IBOR pay-fixed swap (pay fixed, receive IBOR).
- **Short** the OIS pay-fixed swap (so you receive fixed, pay OIS).

Then fixed legs at $K_{\text{rep}}$ cancel, leaving net receive IBOR, pay OIS (a zero-spread basis swap).

**PV of portfolio:**
$PV_{\text{rep}} = PV_{\text{IBORswap}} - PV_{\text{OISswap}} \approx 7{,}155 - (-7{,}375) = \$14{,}530$.

#### E.5 Step 4 — Direct PV of the (No-Spread) Basis Swap

Directly:
$PV_{\text{basis,no-spread}} = \sum_i w_i\bigl(L_i^{\text{IBOR}} - L_i^{\text{OIS}}\bigr)$.

But this numerator was computed in Example B as $\approx 0.014530$ per unit notional, so:
$PV \approx 0.014530 \times 1{,}000{,}000 = \$14{,}530$.

**Match:** $PV_{\text{rep}} = PV_{\text{basis,no-spread}}$ (to rounding). ✅

#### E.6 Adding the Quoted Spread $e$

If the basis swap pays OIS + e, its PV becomes:
$PV_{\text{basis}} = PV_{\text{basis,no-spread}} - N\,W\,e$.

At the par spread from Example B, this equals zero.

---

### Example F — Swap Spread Sensitivity to Benchmark Choice (OTR vs Fitted)

**Goal:** Compute swap spread using two benchmark yields and show the difference.

#### F.1 Conventions

- Maturity: 5Y.
- Swap rate: $S_{\text{swap}} = 4.25\%$.
- Government yield benchmarks:
  - On-the-run yield $y_{\text{OTR}} = 4.00\%$,
  - Fitted curve yield $y_{\text{fit}} = 3.92\%$.
- Swap spread defined as swap rate − government yield.

#### F.2 Calculations

**Using on-the-run:** $\text{SwapSpread}_{\text{OTR}} = 4.25\% - 4.00\% = 0.25\% = 25\text{ bp}$.

**Using fitted curve:** $\text{SwapSpread}_{\text{fit}} = 4.25\% - 3.92\% = 0.33\% = 33\text{ bp}$.

**Difference:**
$33\text{ bp} - 25\text{ bp} = 8\text{ bp}$.

#### F.3 Interpretation

Tuckman warns that using the on-the-run benchmark can be misleading because OTR yields are influenced by liquidity and special financing; a fitted/interpolated yield curve can mitigate this, though it introduces modeling choices.

Therefore, a swap-spread "signal" can move by several bp without economics changing, purely due to benchmark choice.

---

### Example G — Swap Spread RV Trade: DV01-Hedged Construction $"What are you long/short?"$

**Goal:** Build a toy swap-spread trade and DV01-hedge it with a Treasury bond.

#### G.1 Conventions

- Maturity: 5Y.
- Swap leg: pay fixed on a 5Y swap (receive floating).
- Treasury leg: long a 5Y Treasury bond to DV01 hedge.
- Swap notional: $N_{\text{swap}} = \$100{,}000{,}000$.
- Swap fixed payment frequency: annual (toy), $\tau = 1$.

**Discount factors used for swap annuity (toy OIS DFs):**
$P_d(0,1..5) = \{0.97, 0.94, 0.91, 0.88, 0.85\}$.

**Treasury bond characteristics (toy):**
- Clean price $P = 100$ per \$100 face.
- Modified duration $D_{\text{mod}} = 4.60$.
- DV01 formula: $\text{DV01} = P \cdot D_{\text{mod}} / 10{,}000$.

#### G.2 Step 1 — DV01 of the Swap (PV01 to Fixed Rate, Magnitude)

**Swap annuity:**
$A = \sum_{i=1}^{5} \tau P_d(0,i) = 0.97 + 0.94 + 0.91 + 0.88 + 0.85 = 4.55$.

PV change for a 1 bp change in fixed rate (magnitude):
$\text{DV01}_{\text{swap}} \approx N_{\text{swap}} \cdot A \cdot 10^{-4} = 100{,}000{,}000 \times 4.55 \times 10^{-4} = \$45{,}500$.

**Sign check:** for a pay-fixed swap, rates up tends to increase value, so the PV01 sign is $+$ for a +1bp rate shift (toy linear view).

#### G.3 Step 2 — DV01 of the Treasury Bond per \$100 Face

$\text{DV01}_{\text{bond, per \$100}} = \frac{P \cdot D_{\text{mod}}}{10{,}000} = \frac{100 \times 4.60}{10{,}000} = 0.046$.

Thus DV01 per \$1 face is $0.046/100 = 0.00046$.

#### G.4 Step 3 — Hedge Ratio

Let Treasury face amount be $F$ dollars. Then:
$\text{DV01}_{\text{bond}} = 0.00046 \times F$.

DV01 neutrality requires (using the standard hedge-ratio idea):
$$0.00046\,F \approx 45{,}500 \quad\Rightarrow\quad F = \frac{45{,}500}{0.00046} = 98{,}913{,}043 \approx \$98.9\text{ mm face}.$$

**Hedge ratio:**
$h = \frac{F}{N_{\text{swap}}} \approx \frac{98.9}{100} = 0.989$.

#### G.5 "What are you long/short?"

- **Short (swap spread leg):** pay fixed swap (profits if swap rates rise vs funding curve).
- **Long (gov leg):** Treasury bond (profits if gov yields fall).
- **DV01-neutralized:** parallel moves should cancel (approximately), leaving relative move exposure (swap vs Treasury) plus carry/funding.

---

### Example H — Swap Spread Trade P&L Scenarios (Rates, Spread, Funding)

**Goal:** For the Example G position, compute P&L under:
1. parallel rates shift,
2. swap spread change,
3. funding (repo) shock.

#### H.1 Conventions

- Use Example G hedge: pay-fixed swap ($N_{\text{swap}} = \$100\text{mm}$), long Treasury face ($F = \$98.913\text{mm}$).
- Use DV01 approximation:
  - Swap: \$45{,}500/bp (pay-fixed gains when rates rise).
  - Bond: \$45{,}500/bp (long bond loses when yields rise).
- Horizon: $d = 30$ days.
- Treasury bond coupon: $c = 4\%$ annual, paid semiannually (2% per 180 days).
- Repo financing rate: initial $r = 3.50\%$ (Actual/360-like approximation consistent with Tuckman's $rd/360$ term).
- Assume settle on coupon date so $AI(0)=0$ for simplicity.

#### H.2 Scenario 1 — Parallel Shift: Swap Rate +10bp, Treasury Yield +10bp

**Swap P&L:**
\Delta PV_{\text{swap}} \approx +45{,}500 \times 10 = +\$455{,}000.

**Bond price P&L** (long bond loses when yields rise):
$$\Delta PV_{\text{bond}} \approx -45{,}500 \times 10 = -\$455{,}000.$$

**Net (ignoring carry):** $\approx 0$.

**Interpretation:** DV01 hedge worked for parallel moves.

#### H.3 Scenario 2 — Swap Spread Widening: Swap Rate +5bp, Treasury Yield Unchanged

**Swap P&L:**
$$+45{,}500 \times 5 = +\$227{,}500.$$

**Bond price P&L:** $\approx 0$.

**Add carry on the Treasury over 30 days** using Tuckman's decomposition:
$$\text{P\&L} = \text{Price change} + \text{Interest income} - \text{Financing cost}.$$

**Interest income over 30 days** (semiannual coupon accrual):
$$\text{Interest} = F \times 0.02 \times \frac{30}{180} = 98.913\text{ mm} \times 0.0033333 \approx \$329{,}710.$$

**Financing cost:**
$$\text{FinCost} = (F)(rd/360) = 98.913\text{ mm} \times 0.035 \times \frac{30}{360} = 98.913\text{ mm} \times 0.0029167 \approx \$288{,}496.$$

**Carry:**
$$\text{Carry} \approx 329{,}710 - 288{,}496 = +\$41{,}214.$$

**Total scenario-2 P&L (toy):**
$$\boxed{227{,}500 + 41{,}214 = \$268{,}714.}$$

#### H.4 Scenario 3 — Funding/Specialness Shock: Repo Rate +100bp (3.50% → 4.50%), No Price Move

**Swap P&L:** $0$.

**Bond carry under new repo** $r = 4.50\%$:
\text{FinCost} = 98.913\text{ mm} \times 0.045 \times \frac{30}{360} = 98.913\text{ mm} \times 0.00375 \approx \$370{,}924.

Interest income unchanged (\approx \$329{,}710).

**Carry:**
$$\text{Carry} \approx 329{,}710 - 370{,}924 = -\$41{,}214.$$

Relative to the base carry (+\$41{,}214), the funding shock changes P&L by:
$$\boxed{-\$41{,}214 - (+\$41{,}214) = -\$82{,}428.}$$

#### H.5 Interpretation

Even DV01-hedged, a swap-spread package can have meaningful P&L from repo/funding over realistic horizons (and repo specialness can be volatile).

This is why "swap spread" is not a pure derivatives spread: the cash leg drags in financing mechanics.

---

### Example I — Curve RV: DV01-Neutral Steepener (Receive 2Y, Pay 10Y) and Twist P&L

**Goal:** Build a swap-curve steepener and compute P&L under a twist.

#### I.1 Conventions

- Instruments: par swaps on 2Y and 10Y points.
- Use annual-pay toy annuities from OIS discount factors:
$P_d(0,1..10) = \{0.97, 0.94, 0.91, 0.88, 0.85, 0.82, 0.79, 0.76, 0.73, 0.70\}$.

- PV01 of a swap's fixed leg per \\\$1mm notional (magnitude):
$\text{PV01} \approx 1{,}000{,}000 \Bigl(\sum_{i=1}^{n} P_d(0,i)\Bigr) \cdot 10^{-4}$.

**Trade direction (as required):**
- Receive fixed 2Y (long duration).
- Pay fixed 10Y (short duration).
- Notional scaling to DV01-neutralize.

#### I.2 Step 1 — Compute PV01s (per \\\$1mm Notional)

**2Y annuity:**
$A_2 = 0.97 + 0.94 = 1.91 \quad\Rightarrow\quad \text{PV01}_{2Y} = 1{,}000{,}000(1.91) \cdot 10^{-4} = \$191$.

**10Y annuity:**
$A_{10} = 0.97 + 0.94 + 0.91 + 0.88 + 0.85 + 0.82 + 0.79 + 0.76 + 0.73 + 0.70 = 8.35$
$\Rightarrow\quad \text{PV01}_{10Y} = 1{,}000{,}000(8.35) \cdot 10^{-4} = \$835$.

#### I.3 Step 2 — Choose Notionals for DV01-Neutralization

Let 10Y notional be ($N_{10} = \$1$mm), and 2Y notional be ($N_2$). We want:
$N_2(191) \approx N_{10}(835) \quad\Rightarrow\quad N_2 \approx \frac{835}{191} = 4.372\text{ mm}$.

So:
- Receive 2Y fixed on \$4.372mm notional.
- Pay 10Y fixed on \$1.000mm notional.

**Parallel DV01 check (magnitudes):**
- Receive-2Y DV01 (\approx 4.372 \times 191 \approx \$835).
- Pay-10Y DV01 (\approx \$835).
- Net ($\approx 0$). ✅

#### I.4 Step 3 — Twist Scenario P&L

**Twist scenario ("steepening"):**
- 2Y rate increases by $+5$ bp,
- 10Y rate increases by $+20$ bp.

**P&L approximations:**

Receive fixed loses when rates rise:
\Delta PV_{2Y} \approx -(835) \times 5 = -\$4{,}175.

Pay fixed gains when rates rise:
\Delta PV_{10Y} \approx +(835) \times 20 = +\$16{,}700.

**Net:**
\boxed{\Delta PV \approx +\$12{,}525.}

#### I.5 Interpretation

- **DV01-neutral:** limited exposure to parallel moves.
- **Main exposure:** slope factor (10Y–2Y) increasing.

---

### Example J — Curve RV: DV01-Neutral Butterfly (2Y/5Y/10Y) and Curvature Shock

**Goal:** Build a 2–5–10 butterfly and compute P&L under a curvature shock.

#### J.1 Conventions

Use same OIS discount factors as Example I.

**PV01 per \\\$1mm:**
- 2Y: \$191.
- 5Y: (A_5 = 0.97 + 0.94 + 0.91 + 0.88 + 0.85 = 4.55 \Rightarrow \text{PV01}_{5Y} = \$455).
- 10Y: \$835.

**Position structure:**
- Receive fixed 2Y and 10Y (wings),
- Pay fixed 5Y (belly),
- choose belly notional to DV01-neutralize.

#### J.2 Step 1 — Choose Weights

Let $N_2 = N_{10} = \$1$mm. Total wing DV01:
$191 + 835 = 1026$.

Choose belly notional $N_5$ such that:
$N_5(455) \approx 1026 \quad\Rightarrow\quad N_5 = \frac{1026}{455} = 2.255\text{ mm}$.

So the butterfly is:
- Receive 2Y fixed on \\\$1mm.
- Pay 5Y fixed on \$2.255mm.
- Receive 10Y fixed on \\\$1mm.

**DV01 check:**
- Receive wings DV01 $\approx 1026$.
- Pay belly DV01 $\approx 2.255 \times 455 \approx 1026$.
- Net $\approx 0$. ✅

#### J.3 Step 2 — Curvature Shock P&L

**Curvature shock:** belly yield increases $+10$ bp, wings unchanged.

**Pay-fixed 5Y gains:**
\Delta PV_{5Y} \approx +(1026) \times 10 = \$10{,}260.

**Wings unchanged:** $\approx 0$.

**Net:**
\boxed{+\$10{,}260.}

#### J.4 Interpretation

DV01-neutral butterfly isolates relative movement of the belly vs wings, i.e., curvature.

---

### Example K — Carry/Rolldown vs Basis Move for a Basis Trade

**Goal:** Compute deterministic carry/rolldown over a horizon assuming curves unchanged, then apply a basis widening scenario.

#### K.1 Conventions

- Use Example B 2Y semiannual basis swap.
- Notional $N = \$1{,}000{,}000$.
- Position: receive IBOR, pay $OIS + e$ where $e = 0.7548\%$ (par at $t = 0$).
- Horizon: first coupon date ($t = 0.5$ years).
- **Assumption for carry/rolldown:**
  - realized fixings equal initial forwards (toy),
  - discount and projection curves unchanged.

#### K.2 Step 1 — First-Period Realized Net Coupon at $t = 0.5$

**Receive IBOR coupon:**
$CF_{\text{IBOR}} = N\tau L_{0\to0.5}^{\text{IBOR}} = 1{,}000{,}000(0.5)(0.036) = \$18{,}000$.

**Pay OIS + spread coupon:**
$CF_{\text{OIS}+e} = N\tau(L_{0\to0.5}^{\text{OIS}} + e) = 1{,}000{,}000(0.5)(0.030456 + 0.007548)$.

Note $0.030456 + 0.007548 = 0.038004$, so:
$CF_{\text{OIS}+e} = 1{,}000{,}000(0.5)(0.038004) = \$19{,}002$.

**Net cashflow (receive − pay):**
$CF_{0.5} = 18{,}000 - 19{,}002 = -\$1{,}002$.

#### K.3 Step 2 — Mark-to-Market of Remaining Swap at ($t = 0.5$) (Curves Unchanged)

Remaining payment dates: $(1.0, 1.5, 2.0)$.

**Discount factors from (t = 0.5):**
$P(0.5,T) = \frac{P(0,T)}{P(0,0.5)}$.

So:
- (P(0.5,1) = 0.9700/0.9850 = 0.985279)
- (P(0.5,1.5) = 0.9550/0.9850 = 0.969543)
- (P(0.5,2) = 0.9400/0.9850 = 0.954314)

**Weights at $t = 0.5$:** $w_i' = \tau P(0.5,T_i)$:
- $w_1' = 0.5(0.985279) = 0.492640$
- $w_2' = 0.5(0.969543) = 0.484772$
- $w_3' = 0.5(0.954314) = 0.477157$

**Sum** $W' = 1.454568$.

**Compute remaining par spread** (same formula as Example B but on remaining periods):
$e_{\text{rem}} = \frac{\sum w_i'(L_{\text{IBOR}} - L_{\text{OIS}})}{\sum w_i'}$.

Using remaining diffs:
- $0.5 \to 1$: $0.007072$
- $1 \to 1.5$: $0.008584$
- $1.5 \to 2$: $0.009086$

**Numerator:**
$$0.492640(0.007072) + 0.484772(0.008584) + 0.477157(0.009086) \approx 0.011981.$$

Hence:
$e_{\text{rem}} \approx \frac{0.011981}{1.454568} = 0.008237 = 0.8237\% = 82.4\text{ bp}$.

**Value of remaining swap** to the position (receive IBOR, pay OIS+$e_{\text{contract}}$) is:
$$PV_{t=0.5} = N\,(e_{\text{rem}} - e_{\text{contract}})\,W' = 1{,}000{,}000(0.008237 - 0.007548)(1.454568).$$

Difference (= 0.000689). Multiply:
PV_{t=0.5} \approx 1{,}000{,}000(0.000689)(1.454568) \approx \$1{,}002.

#### K.4 Step 3 — Deterministic Carry/Rolldown P&L (Curves Unchanged)

**Total P&L at (t = 0.5)** (realized coupon + MTM of remaining):
$$\text{P\&L} \approx CF_{0.5} + PV_{t=0.5} \approx -1{,}002 + 1{,}002 \approx 0.$$

**Interpretation:** at-par basis swap has ~zero deterministic carry/rolldown in this toy setup.

#### K.5 Step 4 — Basis Widening Scenario (+10bp on Remaining Basis)

At (t = 0.5), suppose IBOR–OIS basis widens by +10 bp for remaining periods (projection change), discount curve unchanged.

**Approximate PV impact on remaining swap:**
$$\Delta PV_{t=0.5} \approx N\,W'\,(0.0010) = 1{,}000{,}000(1.454568)(0.0010) = +\$1{,}455.$$

**Total P&L becomes:**
$$\boxed{\text{P\&L} \approx 0 + 1{,}455 = +\$1{,}455.}$$

**Message:** realized P&L departs from carry/rolldown when the basis moves.

---

### Example L — "What Gets Bumped?": PV01 Under Two Bump Definitions

**Goal:** Recompute a PV01 under:
1. parallel zero-rate bump,
2. par-quote bump with curve rebuild,

and show the difference. This illustrates curve-model dependence (par-point methodology).

#### L.1 Conventions

**Single-curve (legacy-style) toy curve, annual payments.**

**Instruments used to bootstrap:**
- 1Y deposit discount factor $P(0,1) = 0.9700$.
- 2Y par swap rate $S_2 = 3.50\%$.
- 3Y par swap rate $S_3 = 3.80\%$.

**Swap to measure PV01 on:**
- 3Y pay-fixed swap with fixed rate $K = 4.00\%$,
- Notional $N = \$1{,}000{,}000$,
- Annual $\tau = 1$.

**Swap PV under single curve** (derived from standard par logic):
- $PV_{\text{float}} = 1 - P(0,3)$, $PV_{\text{fixed}} = K \sum_{i=1}^{3} P(0,i)$.

(Conceptually consistent with "floating leg at par implies par fixed rate.")

#### L.2 Step 1 — Bootstrap Base Discount Factors $P(0,2), P(0,3)$

**2Y par swap condition** (annual-pay, single curve):
$S_2 = \frac{1 - P_2}{P_1 + P_2} \quad\Rightarrow\quad P_2 = \frac{1 - S_2 P_1}{1 + S_2}$.

Plugging (P_1 = 0.9700), (S_2 = 0.0350):
$P_2 = \frac{1 - 0.035(0.9700)}{1.035} = \frac{0.96605}{1.035} = 0.93338$.

**3Y par swap condition:**
$S_3 = \frac{1 - P_3}{P_1 + P_2 + P_3} \quad\Rightarrow\quad P_3 = \frac{1 - S_3(P_1 + P_2)}{1 + S_3}$.

Compute $P_1 + P_2 = 0.9700 + 0.93338 = 1.90338$. Then:
$P_3 = \frac{1 - 0.038(1.90338)}{1.038} = \frac{1 - 0.072328}{1.038} = \frac{0.927672}{1.038} = 0.89371$.

#### L.3 Step 2 — Base PV of the 3Y Swap (Pay Fixed 4.00%)

**Float PV:**
$PV_{\text{float}} = 1 - P_3 = 1 - 0.89371 = 0.10629$.

**Fixed PV:**
$$PV_{\text{fixed}} = K(P_1 + P_2 + P_3) = 0.04(0.9700 + 0.93338 + 0.89371) = 0.04(2.79709) = 0.11188.$$

**Swap PV (receive float − pay fixed):**
$PV/N = 0.10629 - 0.11188 = -0.00559 \quad\Rightarrow\quad PV \approx -\$5{,}594$.

#### L.4 Method 1 — Parallel Zero-Rate Bump (+1bp), No Rebuild

**Bump rule:**
$P^{+}(0,T) = P(0,T)\,e^{-0.0001\,T}$.

So:
- $P_1^{+} = 0.9700\,e^{-0.0001} \approx 0.969903$
- $P_2^{+} = 0.93338\,e^{-0.0002} \approx 0.933193$
- $P_3^{+} = 0.89371\,e^{-0.0003} \approx 0.893442$

**Recompute PV:**

$$PV_{\text{float}}^{+} = 1 - P_3^{+} = 0.106558.$$

$$\sum P^{+} = 0.969903 + 0.933193 + 0.893442 = 2.796538.$$

$$PV_{\text{fixed}}^{+} = 0.04(2.796538) = 0.111862.$$

$$PV^{+}/N = 0.106558 - 0.111862 = -0.005304 \Rightarrow PV^{+} \approx -\$5{,}304.$$

**So PV01:**
$$\boxed{\text{PV01}_{\text{zero-bump}} = PV^{+} - PV = (-5{,}304) - (-5{,}594) = +\$290 \text{ per \$1mm}.}$$

#### L.5 Method 2 — Par-Quote Bump (+1bp) with Bootstrap Rebuild

This matches the "bump the market quote, rebuild, reprice" idea (par-point approach).

**Bump the market quotes:**

- Bump 1Y deposit rate by +1bp (toy).
  - Base simple 1Y rate $r_1 = \frac{1}{P_1} - 1 = \frac{1}{0.97} - 1 = 0.030928$.
  - Bumped: $r_1^{+} = 0.031028$.
  - New $P_1^{\text{reb}} = \frac{1}{1 + r_1^{+}} \approx \frac{1}{1.031028} \approx 0.9699$.

- Bump par swap quotes:
  - $S_2^{+} = 0.0351$,
  - $S_3^{+} = 0.0381$.

**Rebuild:**
$P_2^{\text{reb}} = \frac{1 - S_2^{+} P_1^{\text{reb}}}{1 + S_2^{+}} \approx 0.93320,\quad P_3^{\text{reb}} = \frac{1 - S_3^{+}(P_1^{\text{reb}} + P_2^{\text{reb}})}{1 + S_3^{+}} \approx 0.89345$.

**Reprice swap:**

$$PV_{\text{float}}^{\text{reb}} = 1 - 0.89345 = 0.10655.$$

$$\sum P^{\text{reb}} \approx 0.9699 + 0.93320 + 0.89345 = 2.79655.$$

$$PV_{\text{fixed}}^{\text{reb}} = 0.04(2.79655) = 0.11186.$$

$$PV^{\text{reb}}/N \approx 0.10655 - 0.11186 = -0.00531 \Rightarrow PV^{\text{reb}} \approx -\$5{,}313.$$

**So PV01:**
$$\boxed{\text{PV01}_{\text{par-bump+rebuild}} = PV^{\text{reb}} - PV = (-5{,}313) - (-5{,}594) = +\$281 \text{ per \$1mm}.}$$

#### L.6 Interpretation

The PV01 differs (\$290 vs \$281 per \$1mm) purely because "1bp shift" was implemented differently.

This is why you must specify "what gets bumped?" and whether curves are rebuilt (par-point approach).

---

## Summary

1. **Every basis trade has a decomposition**: Before entering, articulate precisely what you are long, what you are short, and what residual risks remain after hedging. Use the three-layer framework: level risk, spread risk, residual risk.

2. **Swap spreads are not "pure credit"**: They mix Treasury liquidity/financing (including repo specialness), balance-sheet effects, and convexity-hedging flows. Benchmark choice (on-the-run vs fitted) can contaminate the signal.

3. **The OIS-IBOR basis** separates discounting from projection: Trading it means taking a view on term funding premia and liquidity/credit conditions, not a free arbitrage. The basis can widen materially in stress.

4. **Treasury futures basis** captures delivery option value: The net basis equals the value of the short's delivery options minus any mispricing. P&L follows the simple formula: position size × change in net basis. Near-CTD net basis behaves like a straddle on rates.

5. **Spread of spreads trades** isolate relative value: They profit from convergence between two spreads, not from directional views on either spread individually. Proper construction requires offsetting DV01 exposures.

6. **Basis trades are NOT arbitrage**: LTCM (1998) and later funding/basis stress episodes highlight that convergence trades have significant funding, margin, liquidity, and path risk. Size for the worst-case drawdown, not the expected terminal P&L.

7. **Hidden risks abound**: Funding, roll, liquidity, convexity mismatches, and positioning risk can overwhelm the primary thesis. Understanding these residual exposures is as important as understanding the intended bet.

8. **Multi-curve framework separates discount and projection**: In the post-crisis world, OIS discounting and IBOR projection create separate risk factors. Discount PV01 and projection PV01 must be measured separately to understand where risk lives.

9. **Basis swaps link curves through par conditions**: The quoted basis spread compensates for economic differences between floating indices. Basis PV01 scales with the discounted accrual annuity.

10. **Risk numbers depend on curve construction methodology**: The same "1bp bump" can produce different PV01 depending on whether you bump zeros, forwards, or par quotes, and whether you rebuild curves.

---

## Key Concepts

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **Gross Basis** | $GB^i = P^i - cf^i \times F$ | Starting point for futures basis analysis |
| **Net Basis** | $NB^i = P^i_{fwd} - cf^i \times F = GB^i - carry$ | The actual value traded in a basis position |
| **Basis P&L Formula** | P&L = $G^i \times [NB^i(t') - NB^i(t)]$ | Simple formula for basis trade profit/loss |
| **Swap Spread** | Par swap rate − Treasury yield | Key measure of rates market dynamics, but not purely credit |
| **OIS-IBOR Basis** | Term rate − OIS rate for equivalent tenor | Captures bank credit/liquidity premium in funding |
| **Spread of Spreads** | Difference between two spread measures | Allows trading relative value without directional views |
| **Long/Short Decomposition** | Breaking a trade into its component exposures | Essential for understanding true risk before entry |
| **Quality Option** | Short's option to deliver any eligible bond | Creates CTD dynamics and option-like net basis behavior |
| **CTD** | Cheapest-to-deliver bond | Determines which bond drives futures pricing |
| **Residual Risk** | Exposures remaining after hedging | Often the source of unexpected P&L |
| **Convergence Trade** | Trade that profits from spread narrowing, not at maturity | Basis trades are convergence trades, NOT arbitrage |
| **Multi-Curve Par Rate** | $K_{\text{par}} = \frac{\sum \tau_i P_d L_i}{\sum \tau_i P_d}$ | Discounted average of projection forwards |
| **Basis PV01** | $-N\sum \tau_i P_d \cdot 10^{-4}$ | Sensitivity to quoted basis spread |
| **Discount PV01** | PV sensitivity bumping discount curve only | Measures exposure to OIS/funding curve |
| **Projection PV01** | PV sensitivity bumping projection curve only | Measures exposure to forward rate expectations |

---

## Notation

| Symbol | Definition |
|--------|------------|
| $GB^i(t)$ | Gross basis of bond $i$ at time $t$ |
| $NB^i(t)$ | Net basis of bond $i$ at time $t$ |
| $cf^i$ | Conversion factor for bond $i$ |
| $P^i(t)$ | Spot price of bond $i$ |
| $P^i_{fwd}(t)$ | Forward price of bond $i$ to last delivery |
| $F(t)$ | Futures price |
| $G^i$ | Face amount of bond position |
| $\text{SS}_T$ | Swap spread at maturity $T$ |
| $P_d(0,T)$ | Discount factor to $T$ from discount curve (OIS) |
| $L_k(0,T_i,T_{i+1})$ | Forward rate for index $k$ over $[T_i, T_{i+1}]$ |
| $\tau_i$ | Accrual year fraction for $[T_i, T_{i+1}]$ |
| $N$ | Notional (dollars) |
| $K$ | Fixed rate on a vanilla swap |
| $e_{1,2}(T)$ | Quoted basis spread exchanging $L_1$ vs $L_2$, quoted on the $L_1$ leg |
| $\text{PV01}_{\text{discount}}$ | PV sensitivity to 1 bp bump to discount curve |
| $\text{PV01}_{\text{projection}}$ | PV sensitivity to 1 bp bump to projection curve |
| TED spread | Spread of bond yield to Eurodollar/SOFR futures implied rate |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is the fundamental question before entering any basis trade? | What am I long? What am I short? What residuals remain after hedging? |
| 2 | What is the formula for gross basis in Treasury futures? | $GB^i = P^i - cf^i \times F$ (Spot price minus conversion factor times futures price) |
| 3 | What is the formula for net basis in Treasury futures? | $NB^i = P^i_{fwd} - cf^i \times F$ (Forward price minus CF × futures) = Gross basis − carry |
| 4 | What is the P&L formula for a Treasury futures basis trade? | P&L = $G^i \times [NB^i(t') - NB^i(t)]$, or position size times change in net basis |
| 5 | What does the net basis represent economically? | The value of the quality option with respect to that bond—the cost of committing to deliver that specific bond |
| 6 | How does net basis near CTD behave? | Like a straddle on rates—increases whether rates rise or fall, because any rate move pushes the bond away from CTD |
| 7 | How does net basis of a short-duration bond behave? | Like a call on rates (put on prices)—increases when rates rise |
| 8 | How does net basis of a long-duration bond behave? | Like a put on rates (call on prices)—increases when rates fall |
| 9 | Why is the swap spread not a "pure" measure of bank credit? | It also reflects Treasury supply/demand, repo specialness, regulatory capital, and mortgage hedging flows |
| 10 | What three risk types exist in a multi-curve framework? | Rate level risk, discounting risk, and basis risk |
| 11 | What is a "spread of spreads" trade? | A trade that profits from convergence between two spreads, neutral to the absolute level of either |
| 12 | What does a long basis trade in Treasury futures bet on? | That the delivery options embedded in the futures contract have value that will be realized (net basis widens) |
| 13 | Why did the OIS-IBOR (or LIBOR–OIS) basis widen in 2007–2009 stress? | Term unsecured funding embedded credit/liquidity premia relative to overnight rates; in stress, lenders demand much larger compensation for term bank exposure |
| 14 | Why is using on-the-run Treasuries problematic for swap spread trades? | The on-the-run premium and special repo contaminate the spread; it's not purely about credit |
| 15 | What is "funding risk" in a basis trade? | The risk that financing costs change during the life of the trade, affecting the carry earned |
| 16 | How should a spread of spreads trade be DV01-constructed? | Each leg hedged so the combined DV01 is zero; profits only from spread convergence |
| 17 | What is the "crowded trade" problem? | When many traders have the same position, forced liquidation becomes self-reinforcing |
| 18 | How does the passage of time affect a Treasury futures basis trade? | The net basis must converge to the cost of delivery at expiration |
| 19 | What residual risks remain even in a "DV01-neutral" basis trade? | Curve shape, convexity, funding, roll, and liquidity/positioning risks |
| 20 | What market factor creates "directionality" in swap spreads? | Mortgage hedging: hedgers receive fixed when rates fall (narrowing spreads), pay fixed when rates rise (widening spreads) |
| 21 | Why can quoted swap spreads mislead in detailed trade analysis? | Benchmark choice (OTR vs fitted) and on-the-run liquidity/financing can dominate short-horizon P&L |
| 22 | In the multi-curve world, what do perturbations to basis swap spreads measure? | Basis risk—the risk that index curves of different tenors do not move in lock step |
| 23 | Why might a fundamentally correct basis trade still lose money? | Residual risks (funding, positioning, convexity) can overwhelm the primary thesis during adverse mark-to-market moves |
| 24 | What happened to LTCM in 1998 (high level)? | Flight to quality widened liquidity spreads; leverage + margin calls forced deleveraging before convergence |
| 25 | What is a common failure mode of levered cash–futures basis trades? | Funding/haircuts tighten and futures variation margin drains cash faster than bond P&L can be monetized, forcing deleveraging into a widening basis |
| 26 | What are the four reasons basis trades are NOT arbitrage? | Funding risk, margin risk, CTD risk, liquidity risk (plus path risk to add a fifth) |
| 27 | What is a long basis trade construction? | Buy bond, finance via repo, sell futures (equivalent to long forward, short futures) |
| 28 | What is a short basis trade construction? | Sell bond, invest proceeds, buy futures (equivalent to short forward, long futures) |
| 29 | At delivery, what do gross basis and net basis equal? | The cost of delivery (carry = 0, so GB = NB = cost of delivery) |
| 30 | When is the quality option worthless? | When net basis is near zero—selling the bond forward is equivalent to selling futures |
| 31 | What drives CTD to short-duration bonds? | High yields (above the notional coupon rate) |
| 32 | What drives CTD to long-duration bonds? | Low yields (below the notional coupon rate) |
| 33 | What does a steep yield curve do to CTD? | Richens shorter-duration bonds, pushes CTD toward longer-duration bonds |
| 34 | How large can the LIBOR–OIS spread get in stress? | It can move from single-digit bp in calm markets to hundreds of bp in severe stress (exact peak depends on definition and episode) |
| 35 | What is term repo vs overnight repo? | Term repo locks in financing for a fixed period; overnight repo must be rolled daily (funding cliff risk) |
| 36 | Why is the basis trade "not an arbitrage" despite requiring no net cash outlay? | It abstracts from margin requirements and haircuts—you need capital for margin |
| 37 | What is the "tail" in a basis trade? | Adjustment to futures position for mark-to-market (financing the variation margin) |
| 38 | What is the timing option in Treasury futures? | Short can deliver on any day during the delivery month |
| 39 | What is the end-of-month option? | After last trade date, settlement price is fixed but short can switch bonds if CTD changes |
| 40 | What is the risk-management lesson from convergence-trade blowups? | Stress test “worst of all worlds” paths (spreads widen while funding tightens and margins rise) |
| 41 | What is a TED spread? | The spread such that discounting at Eurodollar rates minus the spread produces the bond's market price |
| 42 | How can a short-basis position cap tail risk? | Use options (or reduce size) to limit losses from large moves / CTD switches |
| 43 | What is the multi-curve par rate formula? | $K_{\text{par}} = \frac{\sum \tau_i P_d L_i}{\sum \tau_i P_d}$ (discounted average of projected forwards) |
| 44 | What is the OIS floating payment formula? | $\displaystyle\frac{R_{01}(0,T) - 1}{\tau}$ where $R_{01} = \prod(1 + f_i \tau_i)$ is the compounded growth factor |
| 45 | How is basis PV01 computed? | $-N\sum \tau_i P_d(0,T_i)\cdot 10^{-4}$ for a position paying the basis spread |
| 46 | What is discount PV01 in the multi-curve framework? | PV change from bumping the OIS discount curve only, holding projection curves fixed |
| 47 | What is projection PV01? | PV change from bumping the IBOR projection curve only, holding discount curve fixed |
| 48 | Why does projection PV01 often dominate discount PV01 for near-par swaps? | Forward bumps change coupon amounts directly; discount bumps rescale both legs and can partially cancel |
| 49 | What is the par condition for a basis swap? | $\sum \tau P_d L_2 = \sum \tau P_d (L_1 + e)$ where $e$ is the quoted spread on the $L_1$ leg |
| 50 | How can you replicate a basis swap with two vanilla swaps? | Long IBOR pay-fixed swap and short OIS pay-fixed swap at same fixed rate; fixed legs cancel |
| 51 | Why do risk numbers depend on "what gets bumped"? | Different bump methodologies (zero-rate bump vs par-quote bump with rebuild) produce different PV01 values |
| 52 | What is a repo-financed bond P&L decomposition? | Price change + interest income − financing cost = price change + carry |
| 53 | What is "specialness" in repo? | When specific collateral financing advantage causes repo rate < general collateral; can be volatile |
| 54 | What is a steepener trade? | Trade that profits when long-term rates rise relative to short-term rates (slope increases); typically receive short-end, pay long-end |
| 55 | What is a butterfly trade? | Trade that isolates curvature; long belly vs short wings (or reverse); DV01-neutral to parallel shifts |
| 56 | What does DV01-neutral mean for curve RV trades? | Net PV01 to parallel shifts is approximately zero; remaining exposure is to curve shape $twist/curvature$ |
| 57 | What is benchmark-choice exposure in swap spreads? | Dependence of measured swap spread on which government yield is used (OTR vs fitted curve) |
| 58 | What links index curves in multi-curve framework? | Floating–floating basis swaps that enforce par conditions tying curves together |
| 59 | Why specify which leg carries the spread in a basis swap? | PV sign conventions and sensitivities depend on which index is the "quoted leg" |
| 60 | What is the key moral of this chapter? | RV trades are never "just a spread"—they embed curve, benchmark, funding, and convexity exposures that must be decomposed |

---

## Mini Problem Set

### Problem 1 (Basic — Definition)
A Treasury futures basis trade has initial net basis of 8 ticks. The trade is for \$50 million face. If the net basis narrows to 3 ticks, what is the P&L for a short basis position?

*Solution:* P&L = \$50MM × (8 - 3)/32 / 100 = \$50MM × 5/3200 = **\$78,125 profit**. (A short basis trader profits when the basis narrows.)

### Problem 2 (Basic — Gross/Net Basis)
A bond has spot price 106-19.5, conversion factor 0.9999, and the futures price is 104-27.5. The carry to delivery is 42.7 ticks. Calculate the gross basis and net basis.

*Solution:*
- Gross basis = 106 + 19.5/32 - 0.9999 × $104 + 27.5/32$ = 106.6094 - 0.9999 × 104.8594 = 1.7605 = **56.3 ticks**
- Net basis = Gross basis - carry = 56.3 - 42.7 = **13.6 ticks**

### Problem 3 (Basic — Swap Spread)
The 10Y swap rate is 4.50% and the 10Y on-the-run Treasury yields 4.05%. What is the swap spread? If the on-the-run trades 10bp special in repo, how does this affect interpretation?

*Solution:* Swap spread = 4.50% - 4.05% = **45bp**. The 10bp repo specialness artificially lowers the Treasury yield (investors accept lower yields for the financing advantage). Adjusting for specialness, the "true" Treasury yield might be closer to 4.15%, giving an adjusted swap spread of about 35bp. The point: quoted swap spreads can be misleading for trade-level analysis.

### Problem 4 (Intermediate — Funding Impact)
You enter a swap spread trade: receive fixed on \$100MM 10Y swap, short \$95MM 10Y Treasury (sized for DV01 match). The Treasury goes very special, costing you 15bp annualized in repo vs. GC. How does this affect your trade over 3 months?

*Solution:* Additional financing cost = \$95MM × 0.15% × (3/12) = **\$35,625**. This is a hidden drag on P&L regardless of what happens to swap spreads. If your spread thesis earns less than this, the trade loses money even if you're "right."

### Problem 5 (Intermediate — Spread of Spreads Sizing)
Bond A has a TED spread of 20bp with DV01 of \$4,500 per \$1MM face. Bond B has a TED spread of 30bp with DV01 of \$5,000 per \$1MM face. You want to trade the spread of spreads with \$10MM of Bond B. How much of Bond A do you need?

*Solution:* Bond B total DV01 = \$10MM × (\$5,000/\$1MM) = \$50,000. To match: need Bond A DV01 = \$50,000. So face = (\$50,000/\$4,500) × \$1MM = **\$11.11MM of Bond A**. You buy \$11.11MM of Bond A (cheap to TED) and sell \$10MM of Bond B (rich to TED).

### Problem 6 (Intermediate — CTD Switch)
TYH2 has three relevant bonds on November 26, 2001:
- 6.00% Aug '09: Net basis 13.6 ticks
- 4.75% Nov '08: Net basis 17.6 ticks
- 5.00% Aug '11: Net basis 34.8 ticks

If yields fall 50bp in parallel, which direction does CTD move? If you were short the net basis in the 6.00s (currently near-CTD), what happens to your P&L?

*Solution:* When yields fall (below notional coupon), longer-duration bonds become relatively cheaper → CTD shifts to longer-duration bonds. The 6.00s move *away* from CTD, and their net basis widens. A short basis position (which profits when net basis narrows) would **lose money**. If net basis widened from 13.6 to, say, 25 ticks on a \$100MM position: Loss = \$100MM × (25 - 13.6)/32 / 100 = **\$356,250**.

### Problem 7 (Intermediate — Multi-Curve Risk)
In the multi-curve framework, a portfolio has:
- \$10MM pay-fixed 5Y swap referencing 3M Term SOFR
- \$10MM receive-fixed 5Y OIS swap

What are the three risk sensitivities (rate level, discounting, basis) of this combined position?

*Solution:*
- **Rate level**: Both swaps have similar DV01 magnitude but opposite signs → approximately zero rate level risk
- **Discounting**: Both are discounted at OIS → remaining sensitivity to OIS curve moves, but largely offset
- **Basis**: The pay-fixed Term SOFR swap is sensitive to the Term SOFR-OIS basis; the receive-fixed OIS swap is not directly exposed. Net position: **long Term SOFR-OIS basis** (you profit if Term SOFR rises relative to OIS).

### Problem 8 (Hard — Curve Risk)
Explain why a DV01-neutral swap spread trade is still exposed to curve steepening/flattening risk. How would you hedge this residual exposure?

*Solution:* The swap and Treasury have different DV01 profiles across the curve (different key rate durations). A swap has distributed coupon exposure; a Treasury has concentrated maturity exposure. A steepening move might benefit the Treasury more than the swap, or vice versa, creating P&L even at zero aggregate DV01. **To hedge:** decompose into key rate DV01 buckets and hedge each, or use additional curve instruments (e.g., a 5s10s steepener/flattener overlay).

### Problem 9 (Hard — Margin Dynamics)
A basis trader is long \$200MM Treasury bonds and short 2,000 TY futures. Rates fall 50bp overnight. The bond gains approximately \$10MM in value, but the futures lose \$10MM. Why might the trader still face a liquidity crisis?

*Solution:* Futures are daily-margined; the \$10MM futures loss requires **immediate cash payment** as variation margin. The bond gain is unrealized until sold or monetized via repo. The trader must fund the margin call from available cash or credit lines. If liquidity is tight or the market is stressed, this asymmetry can force liquidation even though the net P&L is approximately zero.

### Problem 10 (Hard — LTCM Analysis)
Explain why LTCM's "convergence arbitrage" strategy was not true arbitrage. What would have been required for it to be arbitrage?

*Solution:* True arbitrage requires:
1. Guaranteed convergence at a known date
2. No capital required during the trade
3. No risk of forced unwinding before convergence

LTCM's strategy failed these tests:
1. **No guaranteed convergence:** Spreads could widen for extended periods
2. **Capital required:** Margin and collateral were needed; funding was leveraged
3. **Path risk:** Margin calls could force liquidation before convergence

The lesson: **convergence is not arbitrage because the path matters** (funding and margin can force you out before convergence).

### Problem 11 (Hard — Net Basis as Straddle)
Explain why the net basis of a near-CTD bond can behave like a straddle on rates. Use the concept of the quality option.

*Solution:* The net basis equals the value of the quality option with respect to that bond—the cost of committing to deliver it instead of whichever bond is optimal. When your bond is CTD, any move in rates pushes it away from CTD:
- **Rates rise:** Short-duration bonds become cheaper to deliver; your bond $if medium/long duration$ moves away from CTD
- **Rates fall:** Long-duration bonds become cheaper; your bond may still move away from CTD

Either way, the quality option value with respect to your bond *increases*—just like a straddle that profits from volatility in either direction. **A short basis position in a near-CTD bond is implicitly short volatility.**

### Problem 12 (Integration — Case Study)
In Tuckman's November '08 basis trade case study, traders sold net basis at 7.45 ticks and saw it widen to 22 ticks before eventually narrowing to 3.51 ticks. Calculate:
(a) The interim loss at 22 ticks
(b) The final profit at 3.51 ticks
(c) Why might a trader not capture the final profit?

*Solution:*
(a) Interim loss = \$100MM × (22 - 7.45)/32 / 100 = \$100MM × 14.55/3200 = **\$454,688 loss**

(b) Final profit = \$100MM × (7.45 - 3.51)/32 / 100 = \$100MM × 3.94/3200 = **\$123,125 profit**

(c) A trader might not capture the final profit because:
- **Risk limits:** A \$455,000 loss might breach position limits, forcing liquidation
- **Margin calls:** The interim widening requires funding; the trader may lack liquidity
- **Management pressure:** Tuckman notes "a trader showing a loss of $455,000 or $327,219 on this trade might have been ordered to reduce or close the position"
- **Crowded trade:** "many traders were forced to liquidate short basis positions"—forced selling by others made the contract cheaper, extending the drawdown

### Problem 13 $Intermediate — OIS-IBOR Basis Trade$
A trader believes the Term SOFR–OIS basis is too wide at 25bp and will narrow to 15bp. They enter a \$100MM notional 5-year basis swap **paying Term SOFR flat and receiving OIS + 25bp**. If the basis narrows to 15bp after 6 months (with 4.5 years remaining), and the annuity factor for the remaining term is approximately 4.2, what is the approximate mark-to-market gain?

*Solution:* If the fair spread narrows from 25bp to 15bp, a position that receives the spread is receiving 10bp above market on the remaining annuity. Approximate MTM gain:

MTM gain ≈ $100MM × 0.10% × 4.2 = **$420,000**

### Problem 14 (Hard — Carry and Roll in Basis Trades)
A basis trader is short \$50MM net basis at 15 ticks in the TY contract, expecting delivery convergence. The trade has 60 days to expiration. The bond carries at 3 ticks per month (positive carry). If the net basis converges to 2 ticks at delivery (the quality option value at expiration), calculate:
(a) The gross P&L from net basis convergence
(b) The carry earned over 60 days
(c) The total trade P&L

*Solution:*
(a) Net basis P&L = \$50MM × (15 - 2)/32 / 100 = \$50MM × 13/3200 = **\$203,125 profit**

(b) Carry = 3 ticks/month × 2 months = 6 ticks. But carry is already embedded in the net basis formula $net basis = gross basis − carry$. The \$203,125 already accounts for the carry convergence.

(c) Total P&L = **\$203,125** (the carry is not additive; it's the mechanism by which net basis converges)

*Key insight:* The P&L formula $G^i \times [NB^i(t') - NB^i(t)]$ already incorporates carry because net basis is defined as gross basis minus carry. Don't double-count.

### Problem 15 (Hard — Tenor Basis Risk)
A portfolio contains a $200MM 5-year swap that pays 3M Term SOFR quarterly. The swap is hedged with $200MM of 5-year OIS swaps. Both swaps are DV01-matched for parallel rate moves. However, the portfolio has residual exposure to the 3M-OIS tenor basis.

(a) What is the sign of the tenor basis exposure?
(b) If the 3M-OIS basis widens by 10bp, estimate the approximate P&L impact.
(c) How would you hedge this exposure?

*Solution:*
(a) The portfolio pays 3M Term SOFR and receives OIS (net). If the 3M-OIS basis widens (3M rate rises relative to OIS), the portfolio **loses money** because it pays more on the floating leg without receiving more. The position is **short the tenor basis**.

(b) Approximate P&L = $200MM × 0.10% × ~4.5 (annuity factor) ≈ **$900,000 loss**

(c) **Hedge:** Enter a basis swap receiving 3M Term SOFR and paying OIS. This directly offsets the tenor basis exposure. Alternatively, replace some of the OIS hedge with Term SOFR swaps.

### Problem 16 $Integration — Multi-Leg RV Trade$
You want to express the view that 5-year swap spreads are too wide relative to 10-year swap spreads (the "spread of swap spreads" is too high). Design a trade with four legs that:
- Is DV01-neutral overall
- Is DV01-neutral within each maturity
- Profits purely from the spread of swap spreads narrowing

*Solution:*
**Trade Construction:**

| Leg | Instrument | Direction | Purpose |
|-----|------------|-----------|---------|
| 1 | 5Y Treasury | Long | Long 5Y swap spread → short 5Y Treasury |
| 2 | 5Y Swap | Receive fixed | Long 5Y swap spread → receive fixed on swap |
| 3 | 10Y Treasury | Short | Short 10Y swap spread → long 10Y Treasury |
| 4 | 10Y Swap | Pay fixed | Short 10Y swap spread → pay fixed on swap |

**Sizing:**
- Size Legs 1 & 2 to be DV01-neutral at 5Y point
- Size Legs 3 & 4 to be DV01-neutral at 10Y point
- Size the 5Y and 10Y "packages" to have equal and opposite DV01

**Result:**
- Net DV01 = 0 (parallel shifts have no effect)
- 5Y swap spread exposure: Long (profit if 5Y spread narrows)
- 10Y swap spread exposure: Short (profit if 10Y spread widens)
- Net: Profit if 5Y spread narrows *relative to* 10Y spread (spread of spreads compresses)

**Residual risks:** Curve twist (5s10s steepening/flattening), convexity differences between swaps and Treasuries, repo financing on Treasury legs.

### Solution Sketches (Selected)
- Problem 1: +\$78{,}125 profit for short basis when net basis falls 5 ticks: $\$50\text{MM}\times(5/32)/100$.
- Problem 2: Gross basis = 56.3 ticks; net basis = 13.6 ticks.
- Problem 13: +\$420{,}000 (receiving OIS + spread): $\$100\text{MM}\times 10\text{ bp}\times 4.2$.
- Problem 9: Liquidity crisis risk comes from futures variation margin timing vs. monetizing bond P&L (repo terms/haircuts/cash availability), even when net PV is ~0.

---

## References

- Tuckman & Serrat, *Fixed Income Securities*, “Gross and Net Basis”; “Liquidity Premiums of Recent Issues”
- Andersen & Piterbarg, *Interest Rate Modeling*, “Forward Rate Approach” $multi-curve curves; risk sensitivities; Fed funds vs Libor discussion$
- Neftci, *Principles of Financial Engineering*, “Special Versus General Collateral”; “LIBOR and Other Benchmarks”
- Jarrow, *Modeling Fixed Income Securities and Interest Rate Options*, “Treasury Futures Markets” $delivery/wildcard/quality options$
- Hull, *Options, Futures, and Other Derivatives*, “Business Snapshot 2.2 Long-Term Capital Management’s Big Loss”; “Do Not Ignore Liquidity Risk”; “Daily Settlement”; “The Clearing House and Its Members”
- Hull, *Risk Management and Financial Institutions*, “The OIS Rate” (LIBOR–OIS spread as a stress indicator)
- *Market Liquidity Risk*, “2.1 Liquidity and liquidity risk” (market vs funding liquidity)
