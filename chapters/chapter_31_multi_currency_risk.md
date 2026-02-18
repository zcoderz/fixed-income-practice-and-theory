# Chapter 31: Multi-Currency Risk — FX Delta, Rates Delta, Cross-Gamma, and Risk Aggregation

---

## Introduction

When a portfolio contains positions denominated in multiple currencies, the simple question “what is my interest rate risk?” becomes surprisingly complex. You do not just have one yield curve: you have *at least* a curve per currency (and often multiple curves per currency), plus spot and forward FX rates linking currencies, plus (sometimes) a cross-currency basis that reconciles funding/collateral conditions. The key practical consequence is that hedging one bucket (say FX) can create exposure in another (say domestic rates).

A canonical example is a USD investor holding EUR cashflows and hedging EUR/USD with an FX forward. The hedge can neutralize spot FX delta, but the forward’s value depends on *both* domestic and foreign discount factors. That means the FX hedge typically introduces rate DV01 and (depending on the instrument) basis sensitivity. Understanding this plumbing is essential for building hedges that behave as intended and for explaining daily P&L.

Prerequisites: [Chapter 29 (FX spot/forwards)](chapters/chapter_29_fx_spot_forwards.md), [Chapter 30 (FX swaps & XCCY swaps)](chapters/chapter_30_fx_swaps_cross_currency_swaps.md), [Chapter 11 (DV01/PV01)](chapters/chapter_11_dv01_pv01_definitions_computation.md), [Chapter 14 (key-rate DV01)](chapters/chapter_14_key_rate_dv01_bucket_exposures.md), [Chapter 21 (cross-currency curves)](chapters/chapter_21_cross_currency_curves.md).

Follow-on: [Chapter 32 (counterparty exposure)](chapters/chapter_32_counterparty_exposure_basics.md), [Chapter 33 (collateral discounting)](chapters/chapter_33_collateral_discounting_ois.md), [Chapter 34 (XVA overview)](chapters/chapter_34_xva_overview.md).

## Learning Objectives

- Value deterministic multi-currency cashflows in a single reporting currency (and unit-check the result).
- Compute and interpret FX delta and FX01, including common sign traps from quote direction.
- Define per-curve DV01 (discount vs projection) and basis sensitivity with an explicit bump object, size, units, and sign convention.
- Explain (and quantify) how an FX hedge creates rate risk via covered interest parity.
- Recognize when cross-gamma/quanto effects matter and how to sanity-check finite-difference estimates.
- Aggregate risk across books using correlation-based formulas and understand why diversification can fail in stress.

This chapter develops a systematic framework for decomposing and managing multi-currency risk. We cover:

1. **PV in a reporting currency** — converting foreign cashflows to a common base (Section 31.2)
2. **First-order risk decomposition** — separating FX delta, per-curve DV01 (by currency and curve), and basis sensitivity (Section 31.3)
3. **Second-order effects: cross-gamma and quantos** — why large moves and currency mismatches expose nonlinear interactions (Section 31.4)
4. **Risk aggregation** — how correlations affect portfolio VaR, stress scenarios, and when diversification helps (Section 31.5)
5. **Practical hedge instrument mapping** — what to hedge with what, and why hedges create secondary exposures (Section 31.6)
6. **Comprehensive worked examples** — including the global bond fund paradigm and iterative hedging workflows (Section 31.7)
7. **P&L attribution** — decomposing daily P&L across risk factors for multi-currency books (Section 31.8)

The chapter connects to Part IV (curve construction) for multi-curve mechanics, to Chapter 21 for cross-currency basis fundamentals, to Chapter 30 for FX swap and XCCY swap risk decomposition at the instrument level, and previews Part VII’s treatment of counterparty exposure and collateral discounting.

---

## 31.1 Setup: Currencies, Curves, and Bumps

Before diving into mechanics, we establish notation that will be used consistently throughout the chapter.

### 31.1.1 Currency Labels and FX Convention

We designate a **domestic** currency $d$ and a **foreign** currency $f$. The spot FX rate $S_0$ is quoted as:

$$S_0 = \frac{d}{f} \quad \text{(domestic units per one foreign unit)}$$

For example, with $d$ = USD and $f$ = EUR, $S_0 = 1.10$ means 1.10 USD per 1 EUR. Across currency pairs, market quote direction is not uniform; if you invert a quote by mistake you will flip FX delta and hedge ratios.

> **Desk Reality: Quote Direction Traps**
>
> The market quotes EUR/USD as "dollars per euro" (e.g., around 1.10), but USD/JPY as "yen per dollar" (e.g., around 150). A careless sign error when inputting FX delta can mean your "hedge" doubles your risk instead of eliminating it. Always verify: when spot goes *up*, does your domestic-currency P&L go *up* or *down*?

### 31.1.2 Discount Factors and Forward Curves

Following the notation from earlier chapters:

| Symbol | Meaning |
|--------|---------|
| $P_d(0,T)$, $P_f(0,T)$ | Discount factors in domestic and foreign currencies |
| $P_d^{(L)}(0,T)$, $P_f^{(L)}(0,T)$ | Projection/index curves (may differ from discount curves) |
| $L_d(0, t_i, t_{i+1})$, $L_f(0, t_i, t_{i+1})$ | Forward floating rates for accrual period $[t_i, t_{i+1}]$ |
| $e$ | Cross-currency basis spread (quoted; convention-dependent) |

In modern multi-curve setups, **discounting** and **forecasting** can be separated: a discount curve determines present values, while one or more projection curves determine forward floating cashflows. This distinction matters because “rates risk” can mean discount risk, projection risk, or both.

### 31.1.3 Risk Measures

| Symbol | Definition | Units |
|--------|------------|-------|
| $\Delta_{FX}$ | $\partial PV_d / \partial S_0$ | Foreign currency $f$ |
| $FX01$ | $0.01 \cdot S_0 \cdot \Delta_{FX}$ | Domestic currency $d$ per 1% FX move |
| $DV01_{c,\text{disc}}$ | $PV_d(\text{discount zero rates in }c\text{ down }1\text{bp}) - PV_d$ | $d$/bp |
| $DV01_{c,\text{proj}}$ | $PV_d(\text{projection zero rates in }c\text{ down }1\text{bp; discount fixed}) - PV_d$ | $d$/bp |
| $Basis01$ | $PV_d(e\text{ down }1\text{bp}) - PV_d$ (state which leg $e$ applies to) | $d$/bp |

### 31.1.4 Bump Definition for This Chapter

Throughout our examples, we use:

$$P^{\downarrow}(0,T) = P(0,T) \exp(+0.0001 \cdot T)$$

Here $1\text{bp}=10^{-4}$ in rate units. This corresponds to a **-1bp** parallel shift of **continuously-compounded zero rates** (our DV01 convention is “rates down”): if $P(0,T)=e^{-y(0,T)\,T}$, then $y^{\downarrow}(0,T)=y(0,T)-10^{-4}$ implies $P^{\downarrow}(0,T)=P(0,T)e^{+10^{-4}T}$.

Different desks use different bump methodologies (par-quote bumps with curve rebuild, key-rate bumps, which curves are held fixed, etc.). Always record the bump object and rebuild rules before comparing DV01/Basis01 across systems.

**Check (DV01 scale for a single cashflow):** For a deterministic domestic cashflow $C$ at maturity $T$, $PV=C\,P(0,T)$. Under a continuously-compounded zero-rate bump, $\partial P/\partial y = -T\,P$, so
$$\frac{\partial PV}{\partial y}\approx -T\,PV\quad\Rightarrow\quad DV01 \approx PV\times T\times 10^{-4}.$$
This “PV times maturity times 1bp” rule of thumb is a fast magnitude check. Example: a \$100 PV at $T=10$y has $DV01\sim 100\times 10\times 10^{-4}=0.10$ dollars per bp (about 10 bp of DV01 per 1% of PV).

---

## 31.2 PV in a Reporting Currency

### 31.2.1 The Fundamental Decomposition

Consider a portfolio with deterministic cashflows in two currencies: domestic cashflows $\{(C_{d,k}, T_{d,k})\}$ and foreign cashflows $\{(C_{f,j}, T_{f,j})\}$. The present value in domestic currency is:

$$\boxed{PV_d = \sum_k C_{d,k} \, P_d(0, T_{d,k}) \;+\; S_0 \sum_j C_{f,j} \, P_f(0, T_{f,j})}$$

This formula captures the essential structure: domestic cashflows are discounted on the domestic curve, foreign cashflows are discounted on the foreign curve in their native currency, then converted at spot FX.

**Unit check:** The first term has units of $d$ (domestic currency times dimensionless discount factor). The second term has units of $(d/f) \times f = d$. Both terms are in domestic currency, as required.

Equivalently, the domestic value of one foreign zero-coupon bond paying 1 unit of $f$ at $T$ is $S_0\,P_f(0,T)$. Once you accept that statement, the usual no-arbitrage relation for an FX forward (covered interest parity) follows as a ratio of domestic and foreign discount factors (Section 31.3.4).

**Expand (two equivalent PV routes):** A useful consistency check is that you can PV the same foreign cashflow in domestic currency in two equivalent ways:

- **Spot + foreign discounting:** $PV_d = S_0\,P_f(0,T)\,C_f(T)$.
- **Forward + domestic discounting:** $PV_d = P_d(0,T)\,F_{0,T}\,C_f(T)$.

They agree when $F_{0,T}=S_0\,P_f(0,T)/P_d(0,T)$ (covered interest parity). In practice, valuation breaks often come from mixing an FX forward surface and two discount curves that are not mutually consistent under the chosen collateral/basis assumptions.

### 31.2.2 Why Curve Choice Matters

The formula above assumes we know which discount curves to use. In practice, this depends on:

1. **Collateralization status**: A collateralized trade is typically discounted on the OIS curve of the collateral currency
2. **CSA terms**: Different Credit Support Annexes may specify different eligible collateral
3. **Clearing vs bilateral**: Cleared trades have standardized margin arrangements

In practice, the discount curve is usually determined by collateral/CSA terms (collateral currency, margining frequency, etc.) and whether the trade is cleared or bilateral. Without those details you can still learn the mechanics in this chapter, but you should not expect an “exact” numerical PV from incomplete curve assumptions.

---

## 31.3 First-Order Risk Decomposition

### 31.3.1 FX Delta

**Definition:** FX delta measures how PV changes when spot FX moves:

$$\boxed{\Delta_{FX} := \frac{\partial PV_d}{\partial S_0}}$$

For the deterministic cashflow case, differentiating the PV formula with respect to $S_0$:

$$\Delta_{FX} = \sum_j C_{f,j} \, P_f(0, T_{f,j})$$

This equals the present value of foreign cashflows, computed in foreign currency. The intuition is immediate: if spot moves by $dS$, the domestic value of foreign cashflows changes by $dS$ times their foreign PV.

**Units check:** $PV_d$ is in $d$, $S_0$ is in $d/f$, so $\partial PV_d / \partial S_0$ is in $f$. FX delta is denominated in foreign currency.

**Desk-friendly conversion:** The "FX01" reports domestic P&L per 1% spot move:

$$\boxed{FX01 = 0.01 \cdot S_0 \cdot \Delta_{FX}}$$

This converts the foreign-currency sensitivity to domestic-currency P&L for a percentage move rather than an absolute move.

> **Desk Reality: FX Delta in Risk Reports**
>
> When a trader says "I'm long €50mm FX delta," they mean a 1% EUR appreciation generates P&L of approximately $0.01 \times S_0 \times \mathrm{EUR}\,50\text{mm} = 550{,}000$ USD (at $S_0 = 1.10$). Risk limits are often expressed in FX01 rather than raw delta because FX01 is currency-agnostic—it tells you the dollar P&L impact directly.

### 31.3.2 Rates DV01 by Currency and Curve

In a multi-curve world, interest rate sensitivity decomposes along two dimensions: **which currency** and **which curve** (discount vs projection).

**Discount-curve DV01 (currency $c$):**
$$DV01_{c,\text{disc}} := PV_d(\text{discount zero rates in }c\text{ down }1\text{bp}) - PV_d$$

**Projection-curve DV01 (currency $c$, if applicable):**
$$DV01_{c,\text{proj}} := PV_d(\text{projection zero rates in }c\text{ down }1\text{bp; discount fixed}) - PV_d$$

The same definitions apply for domestic ($c=d$) and foreign ($c=f$) curves. For deterministic cashflows, only the relevant **discount** curve enters PV. For floating legs (swaps, XCCY legs), a **projection** curve affects expected coupons, so you can have both discount DV01 and projection DV01.

**Check (derivative form and sign):** With the book convention $DV01 := PV(\text{rates down }1\text{bp})-PV(\text{base})$, a first-order approximation is
$$DV01 \approx -\frac{\partial PV}{\partial y}\times 10^{-4},$$
where $y$ is the bumped continuously-compounded zero rate object. For an option-free long fixed cashflow, $\partial PV/\partial y\lt 0$, so DV01 is positive. For a liability (you pay the cashflow), the sign flips.

> **Pitfall — What is being bumped?:** two systems can both report “DV01” but bump different objects (zero rates vs par rates; with/without curve rebuild; discount vs projection).
> **Why it matters:** hedge ratios become meaningless if you are hedging “DV01-A” with “DV01-B”.
> **Quick check:** write down (1) bump object, (2) bump size (1bp = $10^{-4}$), (3) which curves are rebuilt/held fixed, and (4) DV01 sign convention (“rates down” vs “rates up”).

### 31.3.3 Cross-Currency Basis Risk

Cross-currency (CRX) basis swaps exchange floating payments in two currencies, with a quoted basis spread added to one leg (the quote/sign convention is market-dependent). The quoted basis enters PV **linearly** (holding curves fixed). For example, when the basis $e$ is added to the *foreign* floating leg, the PV contribution from the basis coupons has the form:

$$\text{Basis coupons} \supset -S_0 \cdot e \sum_i \tau_i P_f(0, t_{i+1})$$

**Basis01 (our “spread down” convention):**
$$\boxed{Basis01 := PV_d(e\text{ down }1\text{bp}) - PV_d}$$

For a basis swap where you **pay** “foreign + basis”, increasing the basis makes the swap worse for you ($\partial V/\partial e\lt 0$). Under the “basis down” convention, that means $Basis01$ is typically **positive**: you gain when the basis tightens (moves down).

**Sanity check:** If a +1bp widening hurts you, then a -1bp tightening helps you by roughly the same amount, so $Basis01\gt 0$ is the expected sign.

Because basis quote conventions vary by currency pair and by market, you must state explicitly which leg carries $e$ (and whether $e$ is “added” or “subtracted” in the cashflow definition) before interpreting the sign.

### 31.3.4 Why First-Order Buckets Are Not Independent

A critical insight is that first-order risk buckets—FX delta, per-curve DV01s, and basis sensitivity—are not independent. The cleanest place to see this is the no-arbitrage relation for FX forwards (covered interest parity).

**Anchor (discount-factor form):** the forward FX rate to time $T$ (in domestic-per-foreign units) is
$$\boxed{F_{0,T} = S_0 \frac{P_f(0,T)}{P_d(0,T)}}$$
This is sometimes written as $X_T(0)=X(0)\frac{P_f(0,T)}{P_d(0,T)}$ and is known as the forward FX rate to time $T$.

**Expand (replication intuition):** start with 1 unit of foreign currency.
1. **Foreign path:** invest at $r_f$ to get $e^{r_f T}$ foreign at $T$, and sell that future foreign amount for domestic using an FX forward at $F_{0,T}$.
2. **Domestic path:** convert 1 foreign to $S_0$ domestic now and invest at $r_d$ to get $S_0 e^{r_d T}$ domestic at $T$.

No arbitrage means both paths produce the same domestic amount at $T$, which forces the discount-factor relation above.

**Check (special case + sign/units):** if $P_d(0,T)=e^{-r_d T}$ and $P_f(0,T)=e^{-r_f T}$ for constant continuously-compounded short rates, then $F_{0,T}=S_0 e^{(r_d-r_f)T}$. Also, $S_0$ and $F_{0,T}$ both have units $d/f$. If $r_d\gt r_f$ and the quote is $d/f$, then $F_{0,T}\gt S_0$ (a forward “premium”); if $T\to 0$, then $F_{0,T}\to S_0$.

Because $F_{0,T}$ embeds the **ratio** of discount factors, an FX forward that hedges spot FX delta necessarily inherits interest-rate sensitivity. You can see it directly from the forward PV (Section 31.6.2): it contains a term proportional to $P_d(0,T)$ and a term proportional to $P_f(0,T)$.

---

## 31.4 Second-Order Effects: Cross-Gamma and Quantos

### 31.4.1 Why First-Order Hedging Can Fail

First-order (delta/DV01) hedging assumes that sensitivities are approximately constant. For small moves in a single market variable, this works well. But multi-currency portfolios face two challenges that make first-order hedging inadequate:

1. **Large moves**: When FX moves by 5% or curves shift by 50bp, the linear approximation breaks down
2. **Correlated moves**: FX and rates often move together (e.g., currency weakness accompanied by rising rates during a crisis)

In any nonlinear instrument, second derivatives matter. In the single-underlying case, **gamma** is the rate of change of delta with respect to the underlying price, $\Gamma=\partial^2 \Pi/\partial S^2$. For a delta-neutral portfolio, the leading second-order P&L term under a move $\Delta S$ is approximately:

$$\Delta \Pi \approx \Theta \Delta t + \frac{1}{2} \Gamma \Delta S^2$$

This second-order term can dominate for large moves.

### 31.4.2 Cross-Gamma in Multi-Currency Portfolios

In a multi-currency context, we have multiple market variables: spot FX ($S$), domestic rates ($r_d$), foreign rates ($r_f$), and possibly basis ($e$). The Taylor expansion of portfolio value includes cross-derivative terms:

$$\Delta PV \approx \underbrace{\frac{\partial PV}{\partial S}\Delta S + \frac{\partial PV}{\partial r_d}\Delta r_d + \frac{\partial PV}{\partial r_f}\Delta r_f}_{\text{First-order (delta/DV01)}}$$
$$+ \underbrace{\frac{1}{2}\frac{\partial^2 PV}{\partial S^2}(\Delta S)^2 + \frac{1}{2}\frac{\partial^2 PV}{\partial r_d^2}(\Delta r_d)^2 + \ldots}_{\text{Own second-order (gamma/convexity)}}$$
$$+ \underbrace{\frac{\partial^2 PV}{\partial S \partial r_d}\Delta S \Delta r_d + \frac{\partial^2 PV}{\partial S \partial r_f}\Delta S \Delta r_f + \ldots}_{\text{Cross-gamma terms}}$$

The **cross-gamma** terms capture how FX delta changes when rates move, and vice versa. For a cross-currency swap, these terms can be significant.

### 31.4.3 When Cross-Gamma Matters

Cross-gamma is most important when:

1. **Large positions**: The notional is big enough that second-order effects are material
2. **Correlated stress scenarios**: FX and rates move together (e.g., EM currency crisis with rising local rates)
3. **Nonlinear instruments**: Options on FX or rates, callable cross-currency structures
4. **Long horizons**: The longer the time between hedge rebalancing, the more first-order hedges can drift

Large $|\Gamma|$ means delta drifts quickly, so a “set-and-forget” hedge becomes fragile as either the market moves or time passes.

### 31.4.4 Measuring Cross-Gamma

Cross-gamma can be computed by finite differences:

$$\Gamma_{S,r_d} \approx \frac{PV(S+h_S, r_d+h_r) - PV(S+h_S, r_d) - PV(S, r_d+h_r) + PV(S, r_d)}{h_S \cdot h_r}$$

**Check (bump sizes and numerical noise):** Cross-gamma is a *second derivative*, so it is easy to estimate poorly if bumps are too small (rounding noise) or too large (nonlinear regime).
- A common choice is to set $h_S$ as a small **percentage** of spot (e.g., 10–25 bp of spot, i.e. 0.10%–0.25%), and set $h_r$ as a small **rate** move (e.g., 1 bp or 5 bp).
- Verify scale by checking the cross term in the Taylor expansion: $\Gamma_{S,r}\Delta S\,\Delta r$ should be small for near-linear instruments and grow for options/long horizons.
- Prefer **central differences** if possible (bump up and down) to reduce bias; compare $\Gamma$ computed from $(+h_S,+h_r)$ with $(+h_S,-h_r)$ as a stability check.

In practice, risk systems often report:

- **FX gamma** ($\partial^2 PV / \partial S^2$): how FX delta changes with spot
- **Rates gamma/convexity** ($\partial^2 PV / \partial r^2$): how rates DV01 changes with rates
- **Cross-gamma** ($\partial^2 PV / \partial S \partial r$): the interaction term

Cross-gamma reporting conventions vary across risk systems (spot vs forward bumps, which curve(s) are bumped/rebuilt, bump sizes, and whether other market inputs are held fixed). Always record the bump methodology before comparing cross-gamma numbers across systems.

### 31.4.5 Quanto Derivatives: When Currency Mismatch Affects Pricing

A special case of cross-currency risk arises with **quanto derivatives**—instruments where the underlying is naturally measured in one currency but the payoff is made in another currency at a fixed or pre-specified FX conversion. The buyer is insulated from spot FX moves, but the seller is exposed to the **correlation** between the underlying and FX.

**Example (illustrative):** a USD-settled contract on the Nikkei 225 index is a common example—an underlying naturally measured in JPY with payoff/settlement in USD.

> **Analogy: The Universal Traveler's Magic Wallet**
>
> A **quanto** is like having a "Magic Wallet" for travel:
>
> 1. **Normal Travel (FX Risk):** You go to Japan and buy a ¥10,000 meal. If the yen is strong (100 JPY/USD), that meal costs you $100. If the yen is weak (120 JPY/USD), it costs you $83. Your USD cost depends on FX.
>
> 2. **Quanto Travel (No FX Risk):** With the Magic Wallet, you see a ¥10,000 meal and the wallet instantly debits your bank account \$100 regardless of where JPY/USD is trading.
>
> **The Cost of Magic:** This shield isn't free. If Japanese stock prices (Nikkei) and the yen tend to move together (correlation), the Magic Wallet provider has exposure. They charge you for this correlation risk via the **quanto adjustment**.

### 31.4.6 The Quanto Adjustment: Mathematical Derivation

The key mathematical point is that when you change from a pricing measure associated with one currency to a pricing measure associated with another, the drift of a foreign-currency underlying generally changes. In a common (lognormal) setup, the drift adjustment term takes the form below.

**The Adjustment:** When moving from the currency-$Y$ pricing measure to the currency-$X$ pricing measure, the expected growth rate of $V$ increases by:

$$\boxed{\alpha_V = \rho_{VW} \sigma_V \sigma_W}$$

where $\sigma_V$ is the volatility of $V$, $W$ is the forward exchange rate for maturity $T$ (in a consistent quote convention), $\sigma_W$ is the volatility of $W$, and $\rho_{VW}$ is their correlation.

**Result:** If it is assumed that volatilities and correlation are constant and $E_Y(V_T)$ is the expected value of $V$ at time $T$ in the currency-$Y$ measure, then in the currency-$X$ measure:

$$\boxed{E_X(V_T) = E_Y(V_T) \cdot e^{\rho_{VW} \sigma_V \sigma_W T}}$$

Or as a first-order approximation:

$$E_X(V_T) = E_Y(V_T)(1 + \rho_{VW} \sigma_V \sigma_W T)$$

**Checks:**
- If $\rho=0$, the adjustment factor is 1 (no quanto adjustment).
- If $\rho\lt 0$, the adjustment factor is $\lt 1$ (a downward adjustment).
- Units: $\sigma_V$ and $\sigma_W$ are per $\sqrt{\text{year}}$, so $\rho_{VW}\sigma_V\sigma_W T$ is dimensionless.

**Intuition:** If the underlying $V$ tends to be high precisely when the FX forward $W$ is also high (positive correlation), then (from the payoff-currency viewpoint) “high outcomes” are more valuable than under an independence assumption. The quanto adjustment compensates the seller for that correlation exposure.

> **Worked Example: Quanto Nikkei Futures**
>
> **Setup:** Current Nikkei = 38,000 yen. One-year JPY rate = 0.5%, USD rate = 5%, dividend yield = 1%.
>
> **Standard forward (yen-settled):**
> $$F_Y = 38,000 \times e^{(0.005 - 0.01) \times 1} = 37,810$$
>
> **Quanto adjustment:** Vol of Nikkei = 18%, vol of JPY/USD forward = 10%, correlation = -0.25.
>
> $$\text{Adjustment factor} = e^{(-0.25)(0.18)(0.10)(1)} = e^{-0.0045} = 0.9955$$
>
> **Quanto forward (USD-settled):**
> $$F_X = 37,810 \times 0.9955 = 37,640$$
>
> The negative correlation (Nikkei up when yen weakens) means USD-settled contracts are worth *less* than yen-settled contracts—the adjustment is downward.

> **Desk Reality: Where You See Quantos**
>
> - USD-settled Nikkei 225 futures (USD-settled on a JPY index)
> - Diff swaps (foreign rate applied to domestic principal)
> - Quanto CMS caps/floors (EUR CMS rate, USD payment)
> - Quanto equity options on ADRs
>
> **The hard part:** Estimating $\rho$. Historical correlation may not reflect future correlation, and the correlation itself is not directly observable. This creates **model risk**—the quanto adjustment depends on an input that's uncertain.

---

## 31.5 Risk Aggregation and Correlation

### 31.5.1 The Aggregation Problem

A multi-currency desk may compute separate risk measures for its USD book, EUR book, and cross-currency book. How should these be combined into a total portfolio risk measure?

A common approximation for aggregating these is:

$$\boxed{VaR_{\text{total}} = \sqrt{\sum_i \sum_j VaR_i \, VaR_j \, \rho_{ij}}}$$

where $VaR_i$ is the VaR (or economic-capital estimate) for segment $i$ and $\rho_{ij}$ is the correlation between segment losses. This quadratic form is sometimes called a correlation (or “hybrid”) aggregation approach: it is simple and transparent, but it can understate tail risk when segment losses are skewed/kurtotic or correlations rise in stress.

**Check (limiting cases + correlation sanity):**
- If all segments are perfectly correlated ($\rho_{ij}=1$), then $VaR_{\text{total}}=\sum_i VaR_i$ (no diversification).
- If segments are uncorrelated ($\rho_{ij}=0$ for $i\neq j$), then $VaR_{\text{total}}=\sqrt{\sum_i VaR_i^2}$.
- The correlation matrix $[\rho_{ij}]$ must be symmetric and positive semidefinite; otherwise the square root can produce nonsensical results. In practice, “correlation sets” are often adjusted to enforce consistency (or replaced by factor models).

### 31.5.2 Diversification Effects

When $\rho_{ij} \lt 1$, the total VaR is less than the sum of individual VaRs—this is the **diversification benefit**. Less-than-perfect correlation means part of the risk is diversified away.

**Example:** If two segments have VaRs of \$60mm and \$100mm with correlation 0.4:

$$VaR_{\text{total}} = \sqrt{60^2 + 100^2 + 2 \times 60 \times 100 \times 0.4} = \mathrm{USD}\,135.6\text{mm}$$

This is less than \$160mm (the sum), reflecting diversification.

### 31.5.3 Component VaR and Risk Attribution

**Component VaR** allocates total VaR back to sub-portfolios in a way that is additive:

$$VaR = \sum_{i=1}^{M} C_i$$

where $C_i$ is the component VaR for segment $i$. Euler’s theorem provides the intuition: when the risk measure is treated as a homogeneous function of positions, the total can be written as a sum of components. Component VaRs are therefore a convenient way of allocating a total VaR to sub-portfolios.

For a multi-currency desk, this allows attributing total VaR to:
- USD rates book
- EUR rates book
- FX book
- Cross-currency basis book

**Why this matters:** Component VaR helps identify which segment contributes most to total risk, guiding hedging priorities and limit allocation.

### 31.5.4 Principal Components for Interest Rate Risk

Within each currency, interest rate risk can be further decomposed using **principal components analysis (PCA)** on historical yield-curve changes. PCA produces a small set of orthogonal “shape factors” that explain most of the variance in curve moves.

The key insight from PCA is that most interest rate curve movements can be explained by just two or three factors:

| Factor | Interpretation | Typical Variance Explained |
|--------|----------------|---------------------------|
| PC1 | Parallel shift (level) | ~80-90% |
| PC2 | Twist (slope) | ~5-10% |
| PC3 | Butterfly (curvature) | ~1-3% |

Empirically, it is common to see the first factor explain the vast majority of variance and the first three explain nearly all of it (e.g., on the order of $\sim 85\%$ for PC1 and $\sim 99\%$ for the first three).

**Multi-currency application:** For a portfolio with USD and EUR rate exposure, PCA can be applied separately to each curve. Cross-currency correlation then arises from correlations between the factor scores (level-to-level, slope-to-slope) rather than from individual rate correlations. This is more parsimonious and often more stable.

### 31.5.5 Factor Models for Correlation

A simple one-factor model for correlated variables is:

$$U_i = a_i F + \sqrt{1 - a_i^2} Z_i$$

where $F$ is a common factor, $Z_i$ is an idiosyncratic component, and $a_i$ determines the correlation. This structure implies correlation $\rho_{ij} = a_i a_j$ between variables $i$ and $j$.

For multi-currency risk, one might posit:
- A global rates factor affecting all curves
- Currency-specific factors
- A global FX factor

This reduces the number of correlations to estimate from $n(n-1)/2$ to the number of factor loadings.

### 31.5.6 Correlation Breakdown in Stress: Conceptual Framework

A critical caveat: correlations estimated from normal market conditions may not hold during stress. Even if linear correlations are modest in calm periods, dependence in the tails can be much stronger: many risk factors can move “the wrong way” at the same time.

**Why correlations increase in stress:**

1. **Common factor dominance:** In normal times, idiosyncratic risks matter; in crises, the common "risk-off" factor dominates
2. **Contagion effects:** Problems in one market spread to others as institutions deleverage
3. **Liquidity correlation:** In stress, everything becomes illiquid simultaneously
4. **Behavioral herding:** Investors flee to safety together

A conservative risk-management heuristic is to stress correlations upward (or to use stressed historical windows) rather than relying on calm-period estimates when assessing “how bad can it get?”.

### 31.5.7 Correlation Stress: A Quantitative Example

To illustrate the impact of correlation breakdown, let's revisit the three-desk VaR aggregation from Example G, but now with stressed correlations.

**Normal-market correlations:**
| | USD Rates | EUR Rates | Cross-Currency |
|--|-----------|-----------|----------------|
| USD Rates | 1.00 | 0.65 | 0.30 |
| EUR Rates | 0.65 | 1.00 | 0.45 |
| Cross-Currency | 0.30 | 0.45 | 1.00 |

**Stressed correlations (crisis scenario):**
| | USD Rates | EUR Rates | Cross-Currency |
|--|-----------|-----------|----------------|
| USD Rates | 1.00 | 0.90 | 0.85 |
| EUR Rates | 0.90 | 1.00 | 0.85 |
| Cross-Currency | 0.85 | 0.85 | 1.00 |

**Standalone VaRs:** \$50mm (USD), \$40mm (EUR), \$30mm (Cross-Currency)

**Normal-market aggregated VaR:**
$$VaR_{\text{normal}} = \sqrt{50^2 + 40^2 + 30^2 + 2(50)(40)(0.65) + 2(50)(30)(0.30) + 2(40)(30)(0.45)} = \mathrm{USD}\,97.9\text{mm}$$

**Stressed aggregated VaR:**
$$VaR_{\text{stressed}} = \sqrt{50^2 + 40^2 + 30^2 + 2(50)(40)(0.90) + 2(50)(30)(0.85) + 2(40)(30)(0.85)}$$
$$= \sqrt{2500 + 1600 + 900 + 3600 + 2550 + 2040} = \sqrt{13190} = \mathrm{USD}\,114.8\text{mm}$$

**Impact summary:**

| Metric | Normal Market | Stressed Market | Change |
|--------|---------------|-----------------|--------|
| Aggregated VaR | \$97.9mm | \$114.8mm | +17.3% |
| Sum of standalone | \$120.0mm | \$120.0mm | 0% |
| Diversification benefit | \$22.1mm (18%) | \$5.2mm (4%) | -76% |

**Key insight:** The diversification benefit collapsed from 18% to just 4%. In the limit where all correlations go to 1.0, diversification benefit disappears entirely and aggregated VaR equals the sum of standalones (\$120mm).

> **Desk Reality: Stressed VaR and Correlation Stress**
>
> **Common break:** sizing limits and hedges off calm-period correlations/volatilities, then being surprised when diversification evaporates.
>
> **What to check:** recompute aggregated risk under stressed assumptions—either by (i) using a stressed historical window (for stressed VaR/ES, a bank searches for the 251-day period during which VaR/ES would be greatest), or (ii) explicitly stressing the correlation matrix and re-aggregating.

### 31.5.8 Cross-Currency Wrong-Way Risk

**Wrong-way risk** refers to situations where the probability of default is **positively correlated** with exposure (exposure tends to be high when the counterparty is weak). The opposite case—probability of default negatively correlated with exposure—is often called **right-way risk**.

In a multi-currency context, wrong-way risk takes specific forms:

**Example: EM Sovereign Risk**
Consider a Turkish bank counterparty on a USD/TRY cross-currency basis swap where you receive USD and pay TRY. If Turkey experiences stress:
- TRY weakens significantly (e.g., from 30 to 40 TRY/USD)
- Your swap moves deeply in-the-money (you're owed more USD)
- Simultaneously, Turkish counterparties become more likely to default
- Your exposure is highest precisely when default probability is highest

**Example: Commodity Exporters**
A Brazilian mining company counterparty on FX derivatives hedging their USD export revenues:
- When commodity prices crash, BRL typically weakens
- The mining company loses revenue *and* faces FX derivative losses
- Your exposure increases as their creditworthiness decreases

> **Desk Reality: The EM Dilemma**
>
> Long EM local bonds + short EM currency (via XCCY swap to hedge back to USD) means you're hurt by EM stress on multiple fronts:
> - Local rates rise → bond prices fall
> - Local currency weakens → even after FX hedge, you have basis exposure
> - Counterparty credit deteriorates → if the XCCY is with a local bank
>
> **Practical response:** Banks set different credit limits for "positively correlated" (wrong-way) vs. "negatively correlated" (right-way) exposures. Limits are often tighter when wrong-way dynamics are plausible.

**Connection to Chapter 32:** Full treatment of PFE (Potential Future Exposure) and counterparty risk appears in Part VII. The key point here is that FX exposure and credit risk are not independent—wrong-way risk creates hidden correlations in your overall book.

---

## 31.6 Hedge Instrument Mapping

### 31.6.1 What to Hedge with What

The first-order risk buckets map naturally to hedge instruments:

| Exposure | Hedge Instrument | Rationale |
|----------|------------------|-----------|
| FX delta ($\Delta_{FX}$) | FX spot, forwards, or FX swaps | Forward value changes with spot; delta is the leading exposure |
| Domestic rate DV01 | Domestic swaps, futures, or bonds | Directly sensitive to domestic curve |
| Foreign rate DV01 | Foreign swaps, futures, or bonds | Loads on foreign curve; hedge will also have FX exposure |
| Basis01 | Cross-currency basis swaps | The quoted basis spread $e$ is exactly what basis swaps trade |

### 31.6.2 Why Hedges Create Secondary Exposures

This mapping is not clean in practice. Each hedge instrument brings its own risk profile:

**FX forwards create rate DV01:**
An FX forward at strike $K = F_0 = S_0 P_f/P_d$ has value:
$$V_{\text{fwd}} = N \cdot (K \cdot P_d(0,T) - S_0 \cdot P_f(0,T))$$

Differentiating with respect to domestic and foreign curves shows the forward has both domestic and foreign rate sensitivity.

**Foreign rate hedges create FX exposure:**
If you hedge foreign curve DV01 by entering a foreign-currency swap, the swap itself has value denominated in foreign currency. Its domestic-currency PV depends on spot FX, creating FX delta that must be separately hedged.

**Basis swaps have rate and FX components:**
A cross-currency basis swap combines:
- FX delta (from the notional exchange)
- Domestic and foreign rate DV01 (from the floating legs)
- Basis01 (from the quoted spread)

### 31.6.3 Iterative Hedging in Practice

Because hedges create secondary exposures, practical hedge programs are iterative:

1. **Hedge the largest/most liquid bucket first** (often FX delta)
2. **Measure new exposures** created by the hedge
3. **Hedge remaining exposures** with appropriate instruments
4. **Iterate** until residual exposures are within limits

> **Desk Reality: Why Risk Systems Recompute All Greeks After Each Trade**
>
> When you put on an FX forward to hedge your FX delta, the risk system immediately shows you a *new* rate exposure you didn't have before. This isn't a bug—it's the fundamental interconnectedness of multi-currency risk. Desks often hedge to **tolerance bands** (illustrative: FX delta within ±€5mm, DV01 within ±\$50k/bp) rather than to zero, because:
> 1. Transaction costs of achieving "perfect" hedges exceed the residual risk
> 2. Perfect zeroing may require illiquid instruments
> 3. Hedges may themselves create new risks that require further hedging (infinite regress)

---

## 31.7 Worked Examples

### Baseline Data for Examples

| Parameter | Value |
|-----------|-------|
| Reporting currency $d$ | USD |
| Foreign currency $f$ | EUR |
| Spot $S_0$ | 1.10 USD/EUR |
| $P_d(0,1)$ | 0.97 |
| $P_d(0,2)$ | 0.94 |
| $P_f(0,1)$ | 0.98 |
| $P_f(0,2)$ | 0.95 |
| Discount-factor bump for DV01 | $P^{\downarrow}(0,T) = P(0,T) \exp(+0.0001 \cdot T)$ (rates down 1bp) |

### Example A: Multi-Currency PV Calculation

**Cashflows:**
- Receive +100 USD at $T = 1$
- Receive +80 EUR at $T = 2$

**Step-by-step calculation:**

1. **Domestic PV:** $100 \times 0.97 = 97.00$ USD
2. **Foreign PV (in EUR):** $80 \times 0.95 = 76.00$ EUR
3. **Convert to USD:** $1.10 \times 76.00 = 83.60$ USD
4. **Total:** $97.00 + 83.60 = \mathbf{180.60}$ **USD**

**Unit verification:** All terms are in USD. ✓

### Example B: FX Delta via Finite Difference

**Bump spot by +1%:** $S_1 = 1.10 \times 1.01 = 1.111$

**Reprice:**
$$PV(S_1) = 97.00 + 1.111 \times 76.00 = 97.00 + 84.436 = 181.436 \text{ USD}$$

**Finite-difference FX delta:**
$$\Delta_{FX} = \frac{181.436 - 180.60}{1.111 - 1.10} = \frac{0.836}{0.011} = \mathbf{76.00 \text{ EUR}}$$

This exactly matches the theoretical result: FX delta equals the foreign-currency PV of foreign cashflows.

**FX01:** P&L per 1% move = $0.836$ USD

### Example C: Domestic Discount DV01

**Rates down 1bp at $T = 1$:**
$$P_d^{\downarrow}(0,1) = 0.97 \times e^{+0.0001} \approx 0.970097$$

**Reprice:**
$$PV^{\downarrow} = 100 \times 0.970097 + 83.60 \approx 97.0097 + 83.60 = 180.6097 \text{ USD}$$

**DV01:**
$$DV01_{d,\text{disc}} = PV^{\downarrow}-PV \approx 180.6097 - 180.60 = \mathbf{+0.0097 \text{ USD/bp}}$$

**Sign check:** a long domestic cashflow benefits when domestic rates fall, so DV01 is positive under the “rates down” convention.

### Example D: Foreign Discount DV01

**Rates down 1bp at $T = 2$:**
$$P_f^{\downarrow}(0,2) = 0.95 \times e^{+0.0002} \approx 0.950190$$

**Reprice foreign PV:**
- EUR PV: $80 \times 0.950190 \approx 76.0152$ EUR
- USD equivalent: $1.10 \times 76.0152 \approx 83.6167$ USD
- Total $PV^{\downarrow}$: $97.00 + 83.6167 \approx 180.6167$ USD

**DV01:**
$$DV01_{f,\text{disc}} = PV^{\downarrow}-PV \approx 180.6167 - 180.60 = \mathbf{+0.0167 \text{ USD/bp}}$$

### Worked Example: Hedging a Known EUR Receipt with an FX Forward (and the Induced USD DV01)

**Context**
- You will receive 80 EUR in two years and want to lock its USD value today.
- Goal: neutralize spot FX delta and understand what rate risk the hedge creates.

**Timeline (Make Dates Concrete)**
- Trade date: 2026-02-19
- Spot/valuation date (T+2): 2026-02-23
- Cashflow date and FX forward maturity: 2028-02-23

**Inputs**
- Reporting currency $d$: USD; foreign currency $f$: EUR; spot quote $S_0$ is USD per EUR.
- $S_0 = 1.10$ USD/EUR.
- Discount factors (as of 2026-02-23): $P_d(0,2)=0.94$, $P_f(0,2)=0.95$.
- DV01 bump (this chapter): discount zero rates down 1bp $\Rightarrow$ $P^{\downarrow}(0,T)=P(0,T)e^{+10^{-4}T}$.

**Outputs (What You Produce)**
- PV of the EUR receipt in USD.
- FX delta and FX01.
- Induced USD discount-curve DV01 from the FX forward hedge (units and sign).

**Step-by-step**
1. **PV of the EUR receipt (in USD):**
   - Foreign PV: $80\times P_f(0,2)=80\times 0.95 = 76$ EUR
   - Convert: $PV_d = S_0 \times 76 = 1.10\times 76 = 83.60$ USD
2. **FX delta and FX01:**
   - $\Delta_{FX}=76$ EUR (the foreign-currency PV)
   - $FX01 = 0.01 \times S_0 \times \Delta_{FX} = 0.01\times 1.10\times 76 = 0.836$ USD per 1% move
3. **Hedge FX delta with a forward:**
   - Fair forward: $K = F_{0,2} = S_0 \frac{P_f(0,2)}{P_d(0,2)} \approx 1.1117$
   - Enter a short forward to sell $N=80$ EUR for USD at $K$ on 2028-02-23.
   - Forward FX delta: $\Delta_{FX,\text{fwd}}=-N P_f(0,2)=-80\times 0.95=-76$ EUR, so net FX delta $\approx 0$.
4. **Compute the hedge’s induced USD DV01:**
   - Forward PV: $V_{\text{fwd}} = N(KP_d(0,2) - S_0P_f(0,2)) = 0$ at inception.
   - Rates down 1bp: $P_d(0,2)\to P_d^{\downarrow}(0,2)=0.94e^{+0.0002}\approx 0.940188$.
   - Reprice: $V_{\text{fwd}}^{\downarrow} \approx 80(1.1117\times 0.940188 - 1.10\times 0.95) \approx +0.0167$ USD.
   - Therefore the forward has $DV01_{d,\text{disc}}\approx +0.0167$ USD/bp.

**Cashflows (table)**
| Date | Cashflow | Explanation |
|---|---|---|
| 2028-02-23 | +80 EUR | receipt from the asset |
| 2028-02-23 | -80 EUR | delivery into FX forward |
| 2028-02-23 | $+80K$ USD | USD received from FX forward |

**P&L / Risk Interpretation**
- After hedging, the combined position is economically a USD zero-coupon receipt of $80K$ at 2028-02-23.
- That position has **positive USD DV01** (it gains when USD rates fall) even though the original asset was in EUR.

**Sanity Checks**
- Units: $\Delta_{FX}$ is in EUR, $FX01$ is in USD, and DV01 is in USD/bp.
- Sign: if USD yields rise, the PV of the future USD receipt falls; with $DV01\gt 0$, the approximation $\Delta PV \approx -DV01\times \Delta y_{\text{bp}}$ captures the negative P&L.
- Limit: as $T\to 0$, the induced DV01 goes to 0.

**Debug Checklist (When Your Result Looks Wrong)**
- Hedge notional: did you hedge the *cashflow notional* ($N=80$) rather than the PV (76)?
- Quote direction: are you consistently using USD per EUR (not its inverse)?
- Conventions: are you mixing DV01 (“rates down”) with a PV01 (“rates up”) report?

**References**
- See `## References` for the covered-interest-parity FX forward relation and for the DV01 sign convention used here.

### Example F: Cross-Gamma Scenario Analysis

**Setup:** Same position as Examples A-B (receive 100 USD at T=1, 80 EUR at T=2), but now we consider a joint move in FX and rates.

**Scenario:** EUR weakens by 5% (S falls from 1.10 to 1.045), and foreign rates rise by 25bp simultaneously (a classic EM-style stress).

**Linear (first-order) estimate:**
- FX effect: $-0.05 \times S_0 \times \Delta_{FX} = -0.05 \times 1.10 \times 76 = -4.18$ USD
- Foreign rate effect (yields up): $-DV01_{f,\text{disc}} \times 25 \approx -(+0.0167)\times 25 = -0.42$ USD
- **First-order total:** $-4.60$ USD

**Full repricing:**
- $P_f(0,2) \to 0.95 \times e^{-0.0025 \times 2} = 0.9453$ (25bp bump)
- Foreign PV in EUR: $80 \times 0.9453 = 75.62$ EUR
- Convert at stressed spot: $1.045 \times 75.62 = 79.02$ USD
- Total PV: $97.00 + 79.02 = 176.02$ USD

**Actual loss:** $180.60 - 176.02 = 4.58$ USD

**Cross-gamma contribution:** The actual loss (4.58) is close to the first-order estimate (4.60), with a small difference from:
- Second-order FX gamma: $\frac{1}{2} \Gamma_S (\Delta S)^2$
- Cross-gamma: $\Gamma_{S,r} \Delta S \Delta r$
- Second-order rate convexity: $\frac{1}{2} \Gamma_r (\Delta r)^2$

For this simple deterministic cashflow position, cross-gamma effects are small. For portfolios with options or callable structures, they can be substantial.

### Example G: VaR Aggregation Across Currency Desks

**Setup:** A bank has three desks with independently calculated 1-day 99% VaRs:

| Desk | 1-Day 99% VaR | Correlations |
|------|---------------|--------------|
| USD Rates | \$50mm | — |
| EUR Rates | \$40mm | ρ(USD,EUR) = 0.65 |
| Cross-Currency | \$30mm | ρ(USD,XC) = 0.30, ρ(EUR,XC) = 0.45 |

**Aggregation formula:**
$$VaR_{\text{total}} = \sqrt{\sum_i \sum_j VaR_i VaR_j \rho_{ij}}$$

**Calculation:**
$$= \sqrt{50^2 + 40^2 + 30^2 + 2(50)(40)(0.65) + 2(50)(30)(0.30) + 2(40)(30)(0.45)}$$
$$= \sqrt{2500 + 1600 + 900 + 2600 + 900 + 1080}$$
$$= \sqrt{9580} = \mathbf{\mathrm{USD}\,97.9\text{mm}}$$

**Diversification benefit:** The sum of standalone VaRs is \$120mm. Aggregated VaR is \$97.9mm—a diversification benefit of \$22.1mm (18%).

**Component VaR attribution:**
Using marginal VaR weights, we can decompose the \$97.9mm back to the three desks (component VaRs will sum to total VaR).

### Example H: Cross-Currency Basis Swap Risk Decomposition

**Structure:** Receive USD floating + principal, pay EUR floating + $e$ + principal. Notional = 1 EUR.

**Inputs:**

| Parameter | Value |
|-----------|-------|
| Periods $t_1, t_2$ | 1, 2 years |
| $\tau_1, \tau_2$ | 1, 1 |
| $S_0$ | 1.10 |
| $P_d(1), P_d(2)$ | 0.97, 0.94 |
| $P_f(1), P_f(2)$ | 0.98, 0.95 |
| Basis $e$ | 20bp = 0.0020 |

**USD floating leg PV (notional 1):**
Using forward rates from discount factors (assuming $P^{(L)} = P$):
- $L_d(0,0,1) = 1/0.97 - 1 = 0.0309$
- $L_d(0,1,2) = 0.97/0.94 - 1 = 0.0319$

$$PV_{USD} = 0.0309 \times 0.97 + 0.0319 \times 0.94 + 0.94 \approx 1.0000$$

**EUR leg PV (in EUR):**
- $L_f(0,0,1) = 1/0.98 - 1 = 0.0204$
- $L_f(0,1,2) = 0.98/0.95 - 1 = 0.0316$

$$PV_{EUR} = (0.0204 + 0.002) \times 0.98 + (0.0316 + 0.002) \times 0.95 + 0.95 = 1.0059$$

**Swap PV in USD:**
$$PV = 1.0000 - 1.10 \times 1.0059 = \mathbf{-0.1065 \text{ USD}}$$

**Risk decomposition:**

| Risk Bucket | Calculation | Value |
|-------------|-------------|-------|
| FX delta | $-PV_{EUR}$ | $-1.0059$ EUR |
| Basis01 | $+S_0(\tau_1 P_f(1) + \tau_2 P_f(2)) \times 0.0001$ | $+0.000212$ USD/bp |
| Domestic DV01 | (rates down 1bp; bump and reprice) | $\approx +0.0002$ USD/bp |
| Foreign DV01 | (rates down 1bp; bump and reprice) | $\approx -0.0002$ USD/bp |

### Example I: The Global Bond Fund — FX Hedging Creates Systematic Rate Exposure

This comprehensive example illustrates the central insight that FX-hedged foreign bonds behave as if they have domestic rate exposure.

**Setup:** A USD-based global bond fund holds €500 million face value of German Bunds with 7-year modified duration. The fund policy is to fully hedge FX exposure back to USD.

**Unhedged Position Analysis:**

| Exposure | Calculation | Value |
|----------|-------------|-------|
| Market value (USD) | €500mm × 1.10 | \$550mm |
| EUR duration exposure | €500mm × 7yr | €35mm per 1% rate move |
| FX delta | €500mm (full EUR exposure) | |
| USD rate exposure | None | \$0 |

**FX Hedge Implementation:**

To hedge €500mm FX exposure, the fund sells EUR forward 7 years out.

**Step 1: Forward rate calculation**
Assume USD 7Y discount factor = 0.75, EUR 7Y discount factor = 0.80.
$$F_7 = 1.10 \times \frac{0.80}{0.75} = 1.1733$$

**Step 2: Forward mechanics**
At maturity, the fund will:
- Deliver €500mm (from Bund proceeds)
- Receive \$586.67mm (= €500mm × 1.1733)

**Hedged Position Analysis:**

| Exposure | After FX Hedge |
|----------|----------------|
| FX delta | ≈ 0 (hedged) |
| EUR rate exposure | Still €35mm per 1% |
| **USD rate exposure** | **Now materially positive (positive USD DV01)** |

**Why does USD rate exposure appear?**

The forward's domestic-currency value depends on USD discount factors:
$$V_{\text{fwd}} = N_{EUR} \times (K \cdot P_d(0,7) - S_0 \cdot P_f(0,7))$$

A rise in USD rates reduces $P_d(0,7)$ (say from 0.75 to 0.70), which reduces the PV of the USD you'll receive. The forward therefore has **positive** USD DV01 (it gains when rates fall). A simple approximation is:

$$DV01_{USD,\text{fwd}} \approx N_{EUR} \times K \times P_d(0,7) \times 7 \times 0.0001$$
$$= 500\text{mm} \times 1.1733 \times 0.75 \times 7 \times 0.0001 \approx +\mathrm{USD}\,30{,}800/\text{bp}$$

With this DV01, a +10bp rise in USD yields produces a first-order loss of about $-DV01\times 10\approx -308{,}000$ USD.

**Why this happens intuitively:**

1. The forward locks in a future USD receipt that's valuable *today*
2. That future USD receipt is worth less if USD rates rise (higher discounting)
3. The FX hedge that removed EUR/USD spot risk *created* USD curve risk

> **Desk Reality: The "Global Bond Fund" Problem**
>
> This is why FX-hedged global bond funds behave counterintuitively:
> - When USD rates rise, the fund loses money even if Bund prices are unchanged
> - The fund has **USD DV01** despite owning no USD bonds
> - Portfolio managers must decide: accept the induced USD DV01, or overlay USD rate hedges (e.g., USD swaps)
>
> **Practical response:** Many global fixed income funds use domestic rate overlays (USD swap positions) to manage the induced exposure (“duration neutralization”), subject to mandate and risk limits.

### Example J: Iterative Hedging Workflow (Schematic)

This example shows why multi-currency hedging is often an *iteration* rather than a one-shot calculation.

**Initial position:** a foreign-currency portfolio valued in a single reporting currency.

1. **Compute baseline buckets:** $\Delta_{FX}$ (and $FX01$), $DV01_{c,\text{disc}}$ / $DV01_{c,\text{proj}}$ by curve, and $Basis01$ if applicable.
2. **Hedge FX delta:** use FX forwards (or FX swaps) so that the hedge’s $\Delta_{FX}$ offsets the portfolio’s $\Delta_{FX}$.
3. **Recompute risk:** the FX hedge typically adds domestic and/or foreign DV01 (and sometimes basis sensitivity) because forward prices embed discount factor ratios.
4. **Hedge rates and basis:** use domestic swaps/futures for domestic DV01, foreign swaps/futures for foreign DV01 (noting this can reintroduce FX delta), and XCCY basis swaps for $Basis01$.
5. **Iterate:** repeat until residual exposures are inside risk limits and the hedge set is operationally manageable.

> **Desk Reality: When to Stop Iterating**
>
> Practical considerations that determine the stopping point:
> 1. Transaction costs and liquidity
> 2. Tolerance bands in limits
> 3. Model noise and market-data timing (curves, fixings, “as-of”)
>
> Most desks stop when residuals are comfortably inside limits; the goal is robust hedging, not mathematical perfection.

---

## 31.8 Practical Notes

### 31.8.1 Risk Report Consistency Requirements

A multi-currency risk report must clearly specify:

1. **Reporting currency** and FX quote convention ($S = d/f$ vs $S = f/d$)
2. **Bump definition** for each DV01/Basis01 (bump object + curve rebuild/hold-fixed rules)
3. **Which curves** are bumped for each sensitivity
4. **Basis sign convention** (which leg has the spread, pay vs receive)

Without these specifications, DV01/Basis01 numbers are ambiguous.

### 31.8.2 Common Pitfalls

| Pitfall | Consequence |
|---------|-------------|
| Mixing up $S$ vs $1/S$ | FX delta sign flips |
| Hedging FX but ignoring foreign DV01 | Residual curve risk |
| Assuming "currency hedge = no currency risk" | Basis risk remains |
| Inconsistent curves between pricing and risk | Wrong hedge ratios |
| Ignoring cross-gamma in stress scenarios | Underestimate tail losses |

### 31.8.3 Verification Checks

1. **Unit checks:** Every PV term must end in reporting currency
2. **FX hedge sanity:** After FX hedge, PV sensitivity to spot should be near zero
3. **Small-bump stability:** DV01 stable under 0.5bp vs 1bp bumps
4. **Repricing consistency:** All hedges priced on same curve framework as portfolio

### 31.8.4 P&L Attribution for Multi-Currency Books

Daily P&L attribution decomposes actual P&L into contributions from each risk factor. The Taylor expansion gives us the framework:

Using the chapter’s convention $DV01 := PV(\text{rates down }1\text{bp})-PV(\text{base})$, a realized move of $\Delta y_{\text{bp}}$ (positive when yields rise) contributes approximately $-DV01\times \Delta y_{\text{bp}}$.

$$\boxed{\Delta PV \approx \Delta_{FX} \cdot \Delta S \;-\; DV01_d \cdot \Delta y_{d,\text{bp}} \;-\; DV01_f \cdot \Delta y_{f,\text{bp}} \;-\; Basis01 \cdot \Delta e_{\text{bp}} \;+\; \Gamma\text{-terms} \;+\; \text{Unexplained}}$$

**P&L Attribution Template:**

| Line Item | Formula | Notes |
|-----------|---------|-------|
| FX P&L | $FX01 \times (\%\Delta S)$ | Using FX01 = 0.01 × S × Δ_FX |
| Domestic rates P&L | $-DV01_d \times \Delta y_{d}$ (bp) | Must match the DV01 bump object |
| Foreign rates P&L | $-DV01_f \times \Delta y_{f}$ (bp) | $DV01_f$ is in reporting currency |
| Basis P&L | $-Basis01 \times \Delta e$ (bp) | Must state which leg carries $e$ |
| Gamma/Convexity | $(1/2) \times \Gamma \times (\Delta)^2$ | Often aggregated |
| Cross-gamma | $\Gamma_{S,r} \times \Delta S \times \Delta r$ | Often small |
| Theta/Carry | Time decay, accrual | |
| **Unexplained** | Actual − Attributed | Illustrative target: < 5% of actual |

> **Desk Reality: What Product Control Looks For**
>
> Product controllers investigate unexplained P&L that exceeds materiality thresholds (firm-specific; often expressed as a \$ amount and/or a % of daily P&L). Common causes of unexplained P&L in multi-currency books:
>
> 1. **Curve rebuild timing:** Greek snapshot taken at 4pm, curves rebuilt at 5pm
> 2. **FX rate mismatch:** P&L uses fixing rate, Greeks use mid-rate
> 3. **Settlement effects:** Trade that settled yesterday affects today's accrual
> 4. **Missing cross-gamma:** Large moves where second-order terms matter
> 5. **Basis curve updates:** Basis01 computed on a stale basis curve
>
> The fix is usually to recompute Greeks at the correct time with consistent market data, not to adjust the P&L.

**Worked P&L Attribution Example:**

**Yesterday's Risk:**
- FX delta = +€50mm, S = 1.100
- USD DV01 = +\$25,000/bp
- EUR DV01 = +\$16,500/bp (converted to USD at spot)
- Basis01 = +\$5,000/bp

**Today's Market Moves:**
- EUR/USD: 1.100 → 1.105 (+0.45%)
- USD 5Y: +3bp
- EUR 5Y: -2bp
- Basis: +1bp

**Attributed P&L:**
| Component | Calculation | P&L |
|-----------|-------------|-----|
| FX | 0.01 × 1.10 × €50mm × 0.45 | +\$247,500 |
| USD rates | $-25{,}000 \times 3$ | -\$75,000 |
| EUR rates | $-16{,}500 \times (-2)$ | +\$33,000 |
| Basis | $-5{,}000 \times 1$ | -\$5,000 |
| **Total Attributed** | | **+\$200,500** |

**Actual P&L from Risk System:** +\$195,000

**Unexplained:** -\$5,500 (2.7% — acceptable)

---

## Summary

Multi-currency risk requires systematic decomposition across multiple dimensions:

1. **Choose and document your reporting currency and FX convention**—ambiguity here causes sign errors
2. **PV decomposes cleanly:** domestic PV + (spot × foreign PV)
3. **First-order risks separate into buckets:** FX delta, per-curve DV01s, and basis sensitivity
4. **But hedges create cross-exposures:** an FX forward has rate DV01; a foreign-currency rates hedge has FX delta
5. **FX-hedged foreign bonds create domestic rate exposure**—the "global bond fund" effect
6. **Second-order (cross-gamma) matters for large correlated moves** and option-like structures
7. **Quanto derivatives** have embedded correlation exposure between the underlying and FX
8. **The quanto adjustment is** $E_X(V_T) = E_Y(V_T) \cdot e^{\rho \sigma_V \sigma_W T}$
9. **PCA reduces dimensionality** of rates risk: level/slope/curvature explain ~98% of variance
10. **VaR aggregation uses correlations:** total VaR < sum of standalone VaRs when ρ < 1
11. **Correlations can rise in stress**—diversification benefits can shrink quickly
12. **Wrong-way risk** links exposure and default probability in the same direction
13. **Factor models** provide parsimonious correlation structures for aggregation
14. **Practical hedge programs are iterative:** hedge → measure new exposures → hedge residual → repeat
15. **P&L attribution** decomposes daily P&L into FX, rates, basis, and gamma components

---

## Key Concepts

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| FX Delta | $\partial PV_d / \partial S_0$ | Primary FX exposure measure |
| FX01 | $0.01 \cdot S \cdot \Delta_{FX}$ | Converts to domestic P&L per % move |
| Cross-gamma | $\partial^2 PV / \partial S \partial r$ | Captures FX-rate interaction |
| Basis01 | $PV(e\text{ down }1\text{bp})-PV$ (state which leg carries $e$) | Separate from FX and curve risk |
| VaR aggregation | $\sqrt{\sum_i \sum_j VaR_i VaR_j \rho_{ij}}$ | Captures diversification benefit |
| Component VaR | Marginal contribution to total VaR | Enables risk attribution |
| PCA for rates | Decompose curve moves into level/slope/curvature | Reduces dimensionality; ~98% variance in 3 factors |
| Quanto adjustment | $E_X(V_T) = E_Y(V_T) e^{\rho \sigma_V \sigma_W T}$ | Correlation between underlying and FX affects pricing |
| Factor model | $U_i = a_i F + \sqrt{1-a_i^2} Z_i$ | Parsimonious correlation structure |
| Stressed VaR | VaR computed on a stressed window / stressed assumptions | Captures tail risk when diversification fails |
| Wrong-way risk | Exposure and default likelihood move adversely together | Critical for cross-currency and credit-sensitive counterparties |

---

## Notation

| Symbol | Definition |
|--------|------------|
| $d$, $f$ | Domestic and foreign currency labels |
| $S_0$ | Spot FX rate (domestic per foreign) |
| $F_{0,T}$ | Forward FX rate for settlement at $T$ |
| $P_d(0,T)$, $P_f(0,T)$ | Discount factors in each currency |
| $\Delta_{FX}$ | FX delta (units: foreign currency) |
| $DV01_{c,\text{disc}}$ | $PV(\text{discount zero rates in }c\text{ down }1\text{bp})-PV$ |
| $DV01_{c,\text{proj}}$ | $PV(\text{projection zero rates in }c\text{ down }1\text{bp})-PV$ (discount fixed) |
| $Basis01$ | $PV(e\text{ down }1\text{bp})-PV$ (state which leg carries $e$) |
| $\rho_{ij}$ | Correlation between segments $i$ and $j$ |
| PC1, PC2, PC3 | Principal components (level, slope, curvature) |
| $a_i$ | Factor loading in factor model |
| $F$ | Common factor in factor model |
| $\sigma_V$, $\sigma_W$ | Volatilities of underlying and FX forward (for quantos) |
| $\rho_{VW}$ | Correlation between underlying and FX (for quantos) |
| $\Gamma_{S,r}$ | Cross-gamma (∂²PV/∂S∂r) |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What must be specified first in multi-currency PV? | Reporting currency and FX quote direction |
| 2 | If $S = d/f$, what are the units of FX delta? | Foreign currency $f$ |
| 3 | What is FX01? | P&L per 1% FX move: $0.01 \cdot S \cdot \Delta_{FX}$ |
| 4 | Why do FX hedges create rate DV01? | Forward prices embed discount factor ratios |
| 5 | What is cross-gamma? | Second derivative $\partial^2 PV / \partial S \partial r$ |
| 6 | When does cross-gamma matter most? | Large moves, correlated stress, options |
| 7 | What is the VaR aggregation formula? | $\sqrt{\sum_i \sum_j VaR_i VaR_j \rho_{ij}}$ |
| 8 | When is total VaR < sum of standalone VaRs? | When correlations $\rho_{ij} \lt 1$ |
| 9 | What is component VaR? | Marginal contribution that sums to total VaR |
| 10 | What hedges FX delta? | FX spot, forwards, or FX swaps |
| 11 | What hedges domestic DV01? | Domestic swaps, futures, or bonds |
| 12 | What hedges Basis01? | Cross-currency basis swaps |
| 13 | Why is multi-curve important for risk? | Discount and projection DV01 can differ |
| 14 | What is Basis01? | $PV(e\text{ down }1\text{bp})-PV$ for the stated basis convention |
| 15 | If you pay "foreign + basis," what sign do you expect for Basis01? | Typically positive (you gain when basis tightens) |
| 16 | What is the forward FX formula? | $F_0 = S_0 \cdot P_f(0,T) / P_d(0,T)$ |
| 17 | What is Euler's theorem used for in risk? | Decomposing total VaR into components |
| 18 | Why can correlation breakdown hurt? | Diversification benefit disappears in stress |
| 19 | What is a quanto? | Derivative where underlying is in one currency, payoff in another |
| 20 | What is the quanto adjustment formula? | $E_X(V_T) = E_Y(V_T) e^{\rho \sigma_V \sigma_W T}$ |
| 21 | What are the three main PCA factors for rates? | Level (parallel), slope (twist), curvature (butterfly) |
| 22 | How much variance does PC1 typically explain? | 80-90% (parallel shift) |
| 23 | What is wrong-way risk in FX context? | Exposure increases when counterparty credit deteriorates |
| 24 | Why does FX-hedged foreign bond have domestic rate exposure? | FX forward PV depends on domestic discount factors |
| 25 | What causes secondary exposures when hedging? | Hedge instruments have their own risk profiles |

---

## Mini Problem Set

**Q1 (Compute).** Define $S$ as USD/EUR. If spot moves from 1.10 to 1.12 and your FX delta is +50mm EUR, what is your P&L in USD?

**Q2 (Derive).** Derive the FX delta for a portfolio with deterministic foreign cashflows $\{(C_j,T_j)\}$.

**Q3 (Concept).** Explain why an FX forward that hedges FX delta typically introduces domestic DV01 (state what is held fixed).

**Q4 (Compute).** Two desks have VaRs of \$80mm and \$60mm with correlation 0.5. Calculate total VaR.

**Q5 (Sign).** In Example H you pay “EUR + basis” (basis $e$ is added to the EUR leg). Under $Basis01 := PV(e\text{ down }1\text{bp})-PV$, what sign do you expect and why?

**Q6 (Compute).** Recompute Example G’s aggregated VaR if correlations are 0.9 between all desk pairs.

**Q7 (Desk).** A hedge program neutralizes FX delta but finds residual foreign DV01. Explain why, and what you do next.

**Q8 (Compute).** Quanto adjustment factor for $T=1$: $\sigma_V=25\%$, $\sigma_W=10\%$, $\rho=+0.20$. Compute $e^{\rho\sigma_V\sigma_W T}$.

**Q9 (Compute).** Using the DV01 convention in this chapter: yesterday $\Delta_{FX}=+50\text{mm EUR}$ at $S=1.10$, $DV01_{USD}=+25{,}000\ \mathrm{USD}/\text{bp}$, $DV01_{EUR}=+16{,}500\ \mathrm{USD}/\text{bp}$, $Basis01=+5{,}000\ \mathrm{USD}/\text{bp}$. Today: EUR/USD +0.45%, USD yields +3bp, EUR yields -2bp, basis +1bp. Compute attributed P&L.

**Q10 (Concept).** In a quanto forward, if the correlation between the underlying and FX becomes more positive, what happens to the quanto forward level (qualitatively) and why?

**Q11 (Desk).** Give a concrete example of cross-currency wrong-way risk and one operational check you would make (CSA terms, collateral currency, limits, etc.).

### Solution Sketches (Selected)

- **Q1:** $\Delta PV \approx \Delta S\,\Delta_{FX} = 0.02\times 50\text{mm} = 1.0\text{mm}$ USD.
- **Q3:** Covered interest parity makes the forward PV depend on discount-factor ratios. Holding spot and the foreign curve fixed, bumping the domestic discount curve changes the PV of the domestic cashflow you will receive/pay at maturity, so the hedge carries domestic DV01.
- **Q4:** $\sqrt{80^2 + 60^2 + 2(80)(60)(0.5)} = \sqrt{14800} \approx 121.7\text{mm}$ USD.
- **Q5:** Paying “EUR + basis” means higher $e$ increases your payments so $\partial PV/\partial e\lt 0$. With $Basis01 := PV(e\downarrow 1\text{bp})-PV \approx -\partial PV/\partial e\times 1\text{bp}$, you expect $Basis01\gt 0$.
- **Q9:** FX: $FX01=0.01\times 1.10\times 50\text{mm}=550k$ USD; P&L $\approx 550k\times 0.45=247.5k$ USD. Rates: $-25k\times 3=-75k$ USD; $-16.5k\times (-2)=+33k$ USD. Basis: $-5k\times 1=-5k$ USD. Total $\approx 200.5k$ USD.

---

## References

- Hull, *Options, Futures, and Other Derivatives* (currency forwards; quanto adjustments; option Greeks background)
- Hull, *Risk Management and Financial Institutions* (VaR aggregation, component VaR, stressed VaR intuition)
- Andersen & Piterbarg, *Interest Rate Modeling* (multi-currency discounting relations; cross-currency basis swaps)
- Neftci, *Principles of Financial Engineering* (money-market replication view of FX forwards)
- Tuckman & Serrat, *Fixed Income Securities* (PCA of yield curve moves and factor interpretations)
- McNeil, Frey & Embrechts, *Quantitative Risk Management* (stress dependence and tail dependence intuition)
