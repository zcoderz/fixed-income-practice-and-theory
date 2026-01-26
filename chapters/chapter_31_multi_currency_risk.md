# Chapter 31: Multi-Currency Risk — FX Delta, Rates Delta, Cross-Gamma, and Risk Aggregation

---

## Introduction

When a portfolio contains positions denominated in multiple currencies, the simple question "what is my interest rate risk?" becomes surprisingly complex. A trader holding EUR-denominated bonds hedged with USD interest rate swaps must track not only the PV01 to each yield curve, but also the FX exposure created by the currency mismatch, the cross-currency basis risk embedded in the hedging instruments, and—crucially—the *interaction* between these risk factors when markets move together.

Hull's *Risk Management and Financial Institutions* emphasizes that "interest rate risk is more difficult to manage than the risk arising from market variables such as equity prices, exchange rates, and commodity prices," in part because there are multiple correlated curves in any given currency, and the term structure itself is a function rather than a single number. In a multi-currency setting, this complexity multiplies: we now have term structures in each currency, FX spot and forward rates linking them, and basis spreads reconciling the cross-currency arbitrage relationships.

This chapter develops a systematic framework for decomposing and managing multi-currency risk. We cover:

1. **PV in a reporting currency** — converting foreign cashflows to a common base
2. **First-order risk decomposition** — separating FX delta, rates PV01 (by currency and curve), and basis DV01
3. **Second-order effects: cross-gamma and quantos** — why large moves and currency mismatches expose nonlinear interactions
4. **Risk aggregation** — how correlations affect portfolio VaR, PCA for rates risk, and when diversification helps
5. **Practical hedge instrument mapping** — what to hedge with what, and why hedges create secondary exposures

The chapter connects to Part IV (curve construction) for multi-curve mechanics, to Chapter 21 for cross-currency basis fundamentals, and previews Part VII's treatment of collateral discounting. Throughout, we maintain the discipline of specifying what is source-verified, what is derived, and what remains uncertain.

---

## 31.1 Notation and Conventions

Before diving into mechanics, we establish notation that will be used consistently throughout the chapter.

### 31.1.1 Currency Labels and FX Convention

We designate a **domestic** currency $d$ and a **foreign** currency $f$. The spot FX rate $S_0$ is quoted as:

$$S_0 = \frac{d}{f} \quad \text{(domestic units per one foreign unit)}$$

For example, with $d$ = USD and $f$ = EUR, $S_0 = 1.10$ means 1.10 USD per 1 EUR. Hull explicitly warns about FX quote direction differences across pairs—the same pair might be quoted as EUR/USD or USD/EUR depending on market convention, and misinterpreting this flips the sign of FX delta.

### 31.1.2 Discount Factors and Forward Curves

Following the notation from earlier chapters:

| Symbol | Meaning |
|--------|---------|
| $P_d(0,T)$, $P_f(0,T)$ | Discount factors in domestic and foreign currencies |
| $P_d^{(L)}(0,T)$, $P_f^{(L)}(0,T)$ | Projection/index curves (may differ from discount curves) |
| $L_d(0, t_i, t_{i+1})$, $L_f(0, t_i, t_{i+1})$ | Forward floating rates for accrual period $[t_i, t_{i+1}]$ |
| $e$ | Cross-currency basis spread (quoted; convention-dependent) |

Andersen and Piterbarg note that "the term structure is not a single curve in modern markets: discounting and forecasting can be separated, yielding (at least) a discount curve and one or more forward (projection) curves." This multi-curve framework is essential for understanding curve-specific PV01s.

### 31.1.3 Risk Measures

| Symbol | Definition | Units |
|--------|------------|-------|
| $\Delta_{FX}$ | $\partial PV_d / \partial S_0$ | Foreign currency $f$ |
| $FX01$ | $0.01 \cdot S_0 \cdot \Delta_{FX}$ | Domestic currency $d$ per 1% FX move |
| $PV01_{c,\text{disc}}$ | PV change under +1bp bump to currency $c$'s discount curve | $d$/bp |
| $PV01_{c,\text{proj}}$ | PV change under +1bp bump to currency $c$'s projection curve | $d$/bp |
| $PV01_{\text{basis}}$ | PV change under +1bp move in basis spread $e$ | $d$/bp |

### 31.1.4 Bump Definition for This Chapter

Throughout our examples, we use:

$$P'(0,T) = P(0,T) \exp(-0.0001 \cdot T)$$

This is equivalent to a +1bp parallel shift of continuously-compounded zero rates. Different desks may use different bump conventions (par-quote bumps with curve rebuild, key-rate bumps, etc.); the exact numerical PV01 depends on this choice.

---

## 31.2 PV in a Reporting Currency

### 31.2.1 The Fundamental Decomposition

Consider a portfolio with deterministic cashflows in two currencies: domestic cashflows $\{(C_{d,k}, T_{d,k})\}$ and foreign cashflows $\{(C_{f,j}, T_{f,j})\}$. The present value in domestic currency is:

$$\boxed{PV_d = \sum_k C_{d,k} \, P_d(0, T_{d,k}) \;+\; S_0 \sum_j C_{f,j} \, P_f(0, T_{f,j})}$$

This formula captures the essential structure: domestic cashflows are discounted on the domestic curve, foreign cashflows are discounted on the foreign curve in their native currency, then converted at spot FX.

**Unit check:** The first term has units of $d$ (domestic currency times dimensionless discount factor). The second term has units of $(d/f) \times f = d$. Both terms are in domestic currency, as required.

Andersen and Piterbarg formalize this structure when discussing multi-currency markets: "The value $\widetilde{P}_d$ to a domestic investor of one foreign zero-coupon bond is $\widetilde{P}_d(t,T) = X(t) P_f(t,T)$" where $X(t)$ is the spot exchange rate. The arbitrage-free forward FX rate follows as the ratio of domestic to foreign discount factors.

### 31.2.2 Why Curve Choice Matters

The formula above assumes we know which discount curves to use. In practice, this depends on:

1. **Collateralization status**: A collateralized trade is typically discounted on the OIS curve of the collateral currency
2. **CSA terms**: Different Credit Support Annexes may specify different eligible collateral
3. **Clearing vs bilateral**: Cleared trades have standardized margin arrangements

Hull notes that "after the 2007–08 crisis, institutions switched from LIBOR/swap rates to OIS rates as proxies for risk-free rates" for discounting. I'm not sure about the exact discounting curve choice without knowing your CSA/collateral currency and whether trades are cleared or bilateral.

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

### 31.3.2 Rates PV01 by Currency and Curve

In a multi-curve world, interest rate sensitivity decomposes along two dimensions: **which currency** and **which curve** (discount vs projection).

**Discount curve PV01:**
$$PV01_{d,\text{disc}} := PV_d(P_d \text{ bumped } +1\text{bp}) - PV_d$$

**Projection curve PV01 (if applicable):**
$$PV01_{d,\text{proj}} := PV_d(P_d^{(L)} \text{ bumped } +1\text{bp}, P_d \text{ held fixed}) - PV_d$$

The same definitions apply for foreign currency curves.

Andersen and Piterbarg emphasize that sensitivity measurement should reflect "perturbations to the market instruments used to build curves: funding instruments → discount sensitivity; index instruments → forecasting sensitivity; basis instruments → basis sensitivity." This instrument-based framing is particularly useful for understanding what hedges affect which PV01.

### 31.3.3 Cross-Currency Basis Risk

Cross-currency basis swaps exchange floating payments in two currencies, with a quoted spread added to one leg. Andersen and Piterbarg provide the USD PV of a USD/JPY basis swap (receive USD Libor flat, pay JPY Libor + $e_{¥}$). The basis spread enters the PV linearly:

$$\text{Foreign leg contribution} \supset -X(0) \cdot e \sum_i \tau_i P_f(0, t_{i+1})$$

**Basis DV01:**
$$\boxed{PV01_{\text{basis}} = \frac{\partial V}{\partial e} \times 0.0001}$$

For a basis swap where you pay "foreign + basis," increasing the basis makes the swap worse for you (PV down), so basis DV01 is negative.

**Sanity check:** This matches the formula—if $\partial V / \partial e < 0$, then basis DV01 < 0, meaning you lose when basis widens.

### 31.3.4 Why First-Order Buckets Are Not Independent

A critical insight is that first-order risk buckets—FX delta, domestic PV01, foreign PV01, basis DV01—are not independent of each other. An FX forward used to hedge FX delta also embeds rate PV01 because the forward price depends on both discount curves:

$$F_0 = S_0 \frac{P_f(0,T)}{P_d(0,T)}$$

Differentiating shows the forward has sensitivity to both domestic and foreign discount factors. We will see this explicitly in the worked examples.

---

## 31.4 Second-Order Effects: Cross-Gamma

### 31.4.1 Why First-Order Hedging Can Fail

First-order (delta/DV01) hedging assumes that sensitivities are approximately constant. For small moves in a single market variable, this works well. But multi-currency portfolios face two challenges that make first-order hedging inadequate:

1. **Large moves**: When FX moves by 5% or curves shift by 50bp, the linear approximation breaks down
2. **Correlated moves**: FX and rates often move together (e.g., currency weakness accompanied by rising rates during a crisis)

Hull explains gamma in the single-asset context: "The gamma $(\Gamma)$ of a portfolio of options on an underlying asset is the rate of change of the portfolio's delta with respect to the price of the underlying asset. It is the second partial derivative of the portfolio with respect to asset price." For a delta-neutral portfolio, the P&L under a stock price move $\Delta S$ is approximately:

$$\Delta \Pi \approx \Theta \Delta t + \frac{1}{2} \Gamma \Delta S^2$$

This second-order term can dominate for large moves.

### 31.4.2 Cross-Gamma in Multi-Currency Portfolios

In a multi-currency context, we have multiple market variables: spot FX ($S$), domestic rates ($r_d$), foreign rates ($r_f$), and possibly basis ($e$). The Taylor expansion of portfolio value includes cross-derivative terms:

$$\Delta PV \approx \underbrace{\frac{\partial PV}{\partial S}\Delta S + \frac{\partial PV}{\partial r_d}\Delta r_d + \frac{\partial PV}{\partial r_f}\Delta r_f}_{\text{First-order (delta/PV01)}}$$
$$+ \underbrace{\frac{1}{2}\frac{\partial^2 PV}{\partial S^2}(\Delta S)^2 + \frac{1}{2}\frac{\partial^2 PV}{\partial r_d^2}(\Delta r_d)^2 + \ldots}_{\text{Own second-order (gamma/convexity)}}$$
$$+ \underbrace{\frac{\partial^2 PV}{\partial S \partial r_d}\Delta S \Delta r_d + \frac{\partial^2 PV}{\partial S \partial r_f}\Delta S \Delta r_f + \ldots}_{\text{Cross-gamma terms}}$$

The **cross-gamma** terms capture how FX delta changes when rates move, and vice versa. For a cross-currency swap, these terms can be significant.

### 31.4.3 When Cross-Gamma Matters

Cross-gamma is most important when:

1. **Large positions**: The notional is big enough that second-order effects are material
2. **Correlated stress scenarios**: FX and rates move together (e.g., EM currency crisis with rising local rates)
3. **Nonlinear instruments**: Options on FX or rates, callable cross-currency structures
4. **Long horizons**: The longer the time between hedge rebalancing, the more first-order hedges can drift

Hull notes in the options context: "If gamma is highly negative or highly positive, delta is very sensitive to the price of the underlying asset. It is then quite risky to leave a delta-neutral portfolio unchanged for any length of time."

### 31.4.4 Measuring Cross-Gamma

Cross-gamma can be computed by finite differences:

$$\Gamma_{S,r_d} \approx \frac{PV(S+h_S, r_d+h_r) - PV(S+h_S, r_d) - PV(S, r_d+h_r) + PV(S, r_d)}{h_S \cdot h_r}$$

In practice, risk systems often report:

- **FX gamma** ($\partial^2 PV / \partial S^2$): how FX delta changes with spot
- **Rates gamma/convexity** ($\partial^2 PV / \partial r^2$): how rates PV01 changes with rates
- **Cross-gamma** ($\partial^2 PV / \partial S \partial r$): the interaction term

I'm not sure about the exact cross-gamma reporting conventions across different risk systems without additional specification. The numerical magnitudes depend heavily on the bump sizes used.

### 31.4.5 Quanto Adjustments: When Currency Mismatch Affects Pricing

A special case of cross-currency risk arises with **quanto derivatives**—instruments where the underlying is measured in one currency but the payoff is in another. Hull defines these clearly: "A quanto is a derivative where the underlying asset is measured in one currency and the payoff is in another currency."

**Example:** CME's Nikkei 225 futures contract is a classic quanto—the underlying Nikkei index is denominated in JPY, but the contract settles in USD.

For quantos, the correlation between the underlying and the FX rate affects pricing through a **quanto adjustment**. Hull explains this in the context of diff swaps: "Sometimes a rate observed in one currency is applied to a principal amount in another currency. One such deal would be where a U.S. floating rate is exchanged for a U.K. floating rate with both principals being applied to a principal of 10 million British pounds."

The quanto adjustment arises because when we price a foreign-currency payoff in domestic terms:

> **Analogy: The Universal Traveler**
>
> A **Quanto** is like having a "Magic Wallet" for travel.
>
> 1.  **Normal Travel (FX Risk)**: You go to Japan. You buy a ¥10,000 meal. If the Yen is strong, that meal costs you \$100. If the Yen is weak, it costs you \$80. You care about the exchange rate.
> 2.  **Quanto Travel (No FX Risk)**: You have the Magic Wallet. You see a ¥10,000 meal. The wallet instantly debits your bank account \$100.
>     *   It doesn't matter if the Market FX rate is 80, 100, or 120. Your wallet *always* converts at a fixed rate (e.g., 100).
>
> **The Cost of Magic**: This shield isn't free. If Japanese stock prices (Nikkei) and the Yen are correlated (e.g., stocks go up when Yen goes down), the "Magic Wallet" provider loses money. They charge you for this correlation risk via the **Quanto Adjustment**.
- If the foreign underlying rises and the foreign currency also strengthens (positive correlation), the domestic-currency payoff is amplified
- If they move in opposite directions (negative correlation), the payoff is dampened

This creates a convexity-like effect that must be reflected in the forward price used for pricing. The adjustment is approximately:

$$\text{Quanto Forward} = \text{Standard Forward} \times e^{-\rho_{S,X} \sigma_S \sigma_X T}$$

where $\rho_{S,X}$ is the correlation between the underlying return and the FX rate, and $\sigma_S$, $\sigma_X$ are their volatilities.

**Relevance to risk:** Quanto positions have embedded correlation exposure. A change in the S-X correlation changes the fair value, creating a risk dimension not captured by simple FX delta or rates PV01.

I'm not sure about the exact quanto adjustment formulas for all product types without specifying the underlying model (Hull develops this in the context of specific rate models in Chapter 30).

---

## 31.5 Risk Aggregation and Correlation

### 31.5.1 The Aggregation Problem

A multi-currency desk may compute separate risk measures for its USD book, EUR book, and cross-currency book. How should these be combined into a total portfolio risk measure?

Hull provides the VaR aggregation formula:

$$\boxed{\text{VaR}_{\text{total}} = \sqrt{\sum_i \sum_j \text{VaR}_i \, \text{VaR}_j \, \rho_{ij}}}$$

where $\text{VaR}_i$ is the VaR for segment $i$ and $\rho_{ij}$ is the correlation between losses from segments $i$ and $j$. He notes this "is exactly true when the losses (gains) have zero-mean normal distributions and provides a good approximation in many other situations."

### 31.5.2 Diversification Effects

When $\rho_{ij} < 1$, the total VaR is less than the sum of individual VaRs—this is the **diversification benefit**. Hull illustrates: "The VaR for the combined portfolio is less than the VaR for the Microsoft portfolio plus the VaR for the AT&T portfolio. Less than perfect correlation leads to some of the risk being 'diversified away.'"

**Example:** If two segments have VaRs of \$60mm and \$100mm with correlation 0.4:

$$\text{VaR}_{\text{total}} = \sqrt{60^2 + 100^2 + 2 \times 60 \times 100 \times 0.4} = \$135.6\text{mm}$$

This is less than \$160mm (the sum), reflecting diversification.

### 31.5.3 Component VaR and Risk Attribution

Hull introduces **component VaR** as a way to attribute total portfolio risk to sub-portfolios:

$$\text{VaR} = \sum_{i=1}^{M} C_i$$

where $C_i$ is the component VaR for segment $i$. This uses Euler's theorem: for a linearly homogeneous risk measure, the total equals the sum of partial derivatives times positions.

For a multi-currency desk, this allows attributing total VaR to:
- USD rates book
- EUR rates book
- FX book
- Cross-currency basis book

**Why this matters:** Component VaR helps identify which segment contributes most to total risk, guiding hedging priorities and limit allocation.

### 31.5.4 Principal Components for Interest Rate Risk

When managing multi-currency portfolios, interest rate risk within each currency can be further decomposed using **principal components analysis (PCA)**. Tuckman explains: "Applied to a term structure of interest rates, principal components are a mathematical expression of typical changes in term structure shape as extracted from data on changes in rates."

Hull RM provides a practical interpretation: "One approach to handling the risk arising from groups of highly correlated market variables is principal components analysis... It takes historical data on daily changes in the market variables and attempts to define a set of components or factors that explain the movements."

The key insight from PCA is that most interest rate curve movements can be explained by just two or three factors:

| Factor | Interpretation | Typical Variance Explained |
|--------|----------------|---------------------------|
| PC1 | Parallel shift (level) | ~80-90% |
| PC2 | Twist (slope) | ~5-10% |
| PC3 | Butterfly (curvature) | ~1-3% |

Hull notes: "The first two factors... account for almost all of the variance in the data," showing that "most of the risk in interest rate moves is accounted for by the first two or three factors."

**Multi-currency application:** For a portfolio with USD and EUR rate exposure, PCA can be applied separately to each curve. Cross-currency correlation then arises from correlations between the factor scores (level-to-level, slope-to-slope) rather than from individual rate correlations. This is more parsimonious and often more stable.

### 31.5.5 Factor Models for Correlation

Hull introduces a one-factor model for correlated variables:

$$U_i = a_i F + \sqrt{1 - a_i^2} Z_i$$

where $F$ is a common factor, $Z_i$ is an idiosyncratic component, and $a_i$ determines the correlation. This structure implies correlation $\rho_{ij} = a_i a_j$ between variables $i$ and $j$.

For multi-currency risk, one might posit:
- A global rates factor affecting all curves
- Currency-specific factors
- A global FX factor

This reduces the number of correlations to estimate from $n(n-1)/2$ to the number of factor loadings.

### 31.5.6 Correlation Breakdown in Stress

A critical caveat: correlations estimated from normal market conditions may not hold during stress. McNeil et al. in *Quantitative Risk Management* emphasize "the phenomenon of dependence between extreme outcomes, when many risk factors move against us simultaneously."

When correlations increase toward 1.0 during a crisis (which they tend to do), diversification benefits disappear. A portfolio that looked well-diversified in normal times may show concentrated risk in stress.

**Practical implication:** Stress testing should use correlation matrices calibrated to crisis periods, not normal-market periods. Many institutions run "stressed VaR" scenarios with elevated correlations.

---

## 31.6 Hedge Instrument Mapping

### 31.6.1 What to Hedge with What

The first-order risk buckets map naturally to hedge instruments:

| Exposure | Hedge Instrument | Rationale |
|----------|------------------|-----------|
| FX delta ($\Delta_{FX}$) | FX spot, forwards, or FX swaps | Forward value changes with spot; delta is the leading exposure |
| Domestic rate PV01 | Domestic swaps, futures, or bonds | Directly sensitive to domestic curve |
| Foreign rate PV01 | Foreign swaps, futures, or bonds | Loads on foreign curve; hedge will also have FX exposure |
| Basis DV01 | Cross-currency basis swaps | The quoted basis spread $e$ is exactly what basis swaps trade |

### 31.6.2 Why Hedges Create Secondary Exposures

This mapping is not clean in practice. Each hedge instrument brings its own risk profile:

**FX forwards create rate PV01:**
An FX forward at strike $K = F_0 = S_0 P_f/P_d$ has value:
$$V_{\text{fwd}} = N \cdot (K \cdot P_d(0,T) - S_0 \cdot P_f(0,T))$$

Differentiating with respect to domestic and foreign curves shows the forward has both domestic and foreign rate sensitivity.

**Foreign rate hedges create FX exposure:**
If you hedge foreign curve PV01 by entering a foreign-currency swap, the swap itself has value denominated in foreign currency. Its domestic-currency PV depends on spot FX, creating FX delta that must be separately hedged.

**Basis swaps have rate and FX components:**
A cross-currency basis swap combines:
- FX delta (from the notional exchange)
- Domestic and foreign rate PV01 (from the floating legs)
- Basis DV01 (from the quoted spread)

### 31.6.3 Iterative Hedging in Practice

Because hedges create secondary exposures, practical hedge programs are iterative:

1. **Hedge the largest/most liquid bucket first** (often FX delta)
2. **Measure new exposures** created by the hedge
3. **Hedge remaining exposures** with appropriate instruments
4. **Iterate** until residual exposures are within limits

This is why desks maintain real-time risk systems that recompute all Greeks after each trade.

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
| Bump definition | $P'(0,T) = P(0,T) \exp(-0.0001 \cdot T)$ |

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

### Example C: Domestic Discount PV01

**Bump domestic DF at $T = 1$:**
$$P_d'(0,1) = 0.97 \times e^{-0.0001} = 0.97 \times 0.9999 = 0.969903$$

**Reprice:**
$$PV' = 100 \times 0.969903 + 83.60 = 96.9903 + 83.60 = 180.5903 \text{ USD}$$

**PV01:**
$$PV01_{d,\text{disc}} = 180.5903 - 180.60 = \mathbf{-0.0097 \text{ USD/bp}}$$

The negative sign confirms that higher domestic rates reduce PV (consistent with standard fixed-income intuition).

### Example D: Foreign Discount PV01

**Bump foreign DF at $T = 2$:**
$$P_f'(0,2) = 0.95 \times e^{-0.0002} = 0.95 \times 0.9998 = 0.94981$$

**Reprice foreign PV:**
- EUR PV: $80 \times 0.94981 = 75.9848$ EUR
- USD equivalent: $1.10 \times 75.9848 = 83.5833$ USD
- Total PV': $97.00 + 83.5833 = 180.5833$ USD

**PV01:**
$$PV01_{f,\text{disc}} = 180.5833 - 180.60 = \mathbf{-0.0167 \text{ USD/bp}}$$

### Example E: FX Hedge Creates Rate PV01

**Objective:** Hedge the 80 EUR receipt at $T = 2$ using an FX forward.

**Step 1: Calculate fair forward rate**
$$K = F_{0,2} = S_0 \frac{P_f(0,2)}{P_d(0,2)} = 1.10 \times \frac{0.95}{0.94} = \mathbf{1.1117}$$

**Step 2: Enter short forward (sell 80 EUR at $K$)**

The forward has FX delta:
$$\Delta_{FX,\text{fwd}} = -N \cdot P_f(0,2) = -80 \times 0.95 = -76 \text{ EUR}$$

**Net FX delta after hedge:** $+76 - 76 = 0$ EUR ✓

**Step 3: Check rate PV01 of the hedge**

The forward value is:
$$V_{\text{fwd}} = 80 \times (K \cdot P_d(0,2) - S_0 \cdot P_f(0,2)) = 80 \times (1.1117 \times 0.94 - 1.10 \times 0.95) = 0$$

Under +1bp domestic bump ($P_d(0,2) \to 0.939812$):
$$V_{\text{fwd}}' = 80 \times (1.1117 \times 0.939812 - 1.10 \times 0.95) = 80 \times (1.0448 - 1.045) = -0.0167 \text{ USD}$$

**Forward's domestic PV01:** $-0.0167$ USD/bp

**Key insight:** The FX hedge neutralized FX delta but introduced domestic rate PV01. This is unavoidable because forward prices embed interest rate differentials.

### Example F: Cross-Gamma Scenario Analysis

**Setup:** Same position as Examples A-B (receive 100 USD at T=1, 80 EUR at T=2), but now we consider a joint move in FX and rates.

**Scenario:** EUR weakens by 5% (S falls from 1.10 to 1.045), and foreign rates rise by 25bp simultaneously (a classic EM-style stress).

**Linear (first-order) estimate:**
- FX effect: $-0.05 \times S_0 \times \Delta_{FX} = -0.05 \times 1.10 \times 76 = -4.18$ USD
- Foreign rate effect: $25 \times PV01_{f,\text{disc}} = 25 \times (-0.0167) = -0.42$ USD
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

**Aggregation formula (Hull):**
$$\text{VaR}_{\text{total}} = \sqrt{\sum_i \sum_j \text{VaR}_i \text{VaR}_j \rho_{ij}}$$

**Calculation:**
$$= \sqrt{50^2 + 40^2 + 30^2 + 2(50)(40)(0.65) + 2(50)(30)(0.30) + 2(40)(30)(0.45)}$$
$$= \sqrt{2500 + 1600 + 900 + 2600 + 900 + 1080}$$
$$= \sqrt{9580} = \mathbf{\$97.9\text{mm}}$$

**Diversification benefit:** The sum of standalone VaRs is \$120mm. Aggregated VaR is \$97.9mm—a diversification benefit of \$22.1mm (18%).

**Component VaR attribution:**
Using marginal VaR weights, we can decompose the \$97.9mm back to the three desks (component VaRs will sum to total VaR).

### Example H: Cross-Currency Basis Swap Risk Decomposition

**Structure (per Andersen & Piterbarg):** Receive USD floating + principal, pay EUR floating + $e$ + principal. Notional = 1 EUR.

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
| Basis DV01 | $-S_0(\tau_1 P_f(1) + \tau_2 P_f(2)) \times 0.0001$ | $-0.000212$ USD/bp |
| Domestic PV01 | (bump and reprice) | $\approx -0.0002$ USD/bp |
| Foreign PV01 | (bump and reprice) | $\approx +0.0002$ USD/bp |

---

## 31.8 Practical Notes

### 31.8.1 Risk Report Consistency Requirements

A multi-currency risk report must clearly specify:

1. **Reporting currency** and FX quote convention ($S = d/f$ vs $S = f/d$)
2. **Bump definition** for each PV01 (zero bump vs par bump/rebuild)
3. **Which curves** are bumped for each sensitivity
4. **Basis sign convention** (which leg has the spread, pay vs receive)

Without these specifications, PV01 numbers are ambiguous.

### 31.8.2 Common Pitfalls

| Pitfall | Consequence |
|---------|-------------|
| Mixing up $S$ vs $1/S$ | FX delta sign flips |
| Hedging FX but ignoring foreign PV01 | Residual curve risk |
| Assuming "currency hedge = no currency risk" | Basis risk remains |
| Inconsistent curves between pricing and risk | Wrong hedge ratios |
| Ignoring cross-gamma in stress scenarios | Underestimate tail losses |

### 31.8.3 Verification Checks

1. **Unit checks:** Every PV term must end in reporting currency
2. **FX hedge sanity:** After FX hedge, PV sensitivity to spot should be near zero
3. **Small-bump stability:** PV01 stable under 0.5bp vs 1bp bumps
4. **Repricing consistency:** All hedges priced on same curve framework as portfolio

---

## Summary

Multi-currency risk requires systematic decomposition across multiple dimensions:

1. **Choose and document your reporting currency and FX convention**—ambiguity here causes sign errors
2. **PV decomposes cleanly:** domestic PV + (spot × foreign PV)
3. **First-order risks separate into buckets:** FX delta, domestic PV01, foreign PV01, and basis DV01
4. **But hedges create cross-exposures:** an FX forward has rate PV01; a foreign swap has FX delta
5. **Second-order (cross-gamma) matters for large correlated moves** and option-like structures
6. **Quanto derivatives** have embedded correlation exposure between the underlying and FX
7. **PCA reduces dimensionality** of rates risk: level/slope/curvature explain ~98% of variance
8. **VaR aggregation uses correlations:** total VaR < sum of standalone VaRs when ρ < 1
9. **Factor models** provide parsimonious correlation structures for aggregation
10. **Correlation breakdown in stress** can eliminate diversification benefits
11. **Practical hedge programs are iterative:** hedge → measure new exposures → hedge residual → repeat

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| FX Delta | $\partial PV_d / \partial S_0$ | Primary FX exposure measure |
| FX01 | $0.01 \cdot S \cdot \Delta_{FX}$ | Converts to domestic P&L per % move |
| Cross-gamma | $\partial^2 PV / \partial S \partial r$ | Captures FX-rate interaction |
| Basis DV01 | Sensitivity to quoted basis spread | Separate from FX and curve risk |
| VaR aggregation | $\sqrt{\sum_i \sum_j VaR_i VaR_j \rho_{ij}}$ | Captures diversification benefit |
| Component VaR | Marginal contribution to total VaR | Enables risk attribution |
| PCA for rates | Decompose curve moves into level/slope/curvature | Reduces dimensionality; ~98% variance in 3 factors |
| Quanto adjustment | Correction for currency mismatch in derivatives | Correlation between underlying and FX affects pricing |
| Factor model | $U_i = a_i F + \sqrt{1-a_i^2} Z_i$ | Parsimonious correlation structure |

---

## Notation for This Chapter

| Symbol | Definition |
|--------|------------|
| $d$, $f$ | Domestic and foreign currency labels |
| $S_0$ | Spot FX rate (domestic per foreign) |
| $F_{0,T}$ | Forward FX rate for settlement at $T$ |
| $P_d(0,T)$, $P_f(0,T)$ | Discount factors in each currency |
| $\Delta_{FX}$ | FX delta (units: foreign currency) |
| $PV01_{c,\text{disc}}$ | Discount curve PV01 for currency $c$ |
| $PV01_{\text{basis}}$ | Basis DV01 to quoted spread $e$ |
| $\rho_{ij}$ | Correlation between segments $i$ and $j$ |
| PC1, PC2, PC3 | Principal components (level, slope, curvature) |
| $a_i$ | Factor loading in factor model |
| $F$ | Common factor in factor model |
| $\sigma_S$, $\sigma_X$ | Volatilities of underlying and FX rate (for quantos) |
| $\rho_{S,X}$ | Correlation between underlying and FX (for quantos) |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What must be specified first in multi-currency PV? | Reporting currency and FX quote direction |
| 2 | If $S = d/f$, what are the units of FX delta? | Foreign currency $f$ |
| 3 | What is FX01? | P&L per 1% FX move: $0.01 \cdot S \cdot \Delta_{FX}$ |
| 4 | Why can FX hedges create rate PV01? | Forward prices embed discount factor ratios |
| 5 | What is cross-gamma? | Second derivative $\partial^2 PV / \partial S \partial r$ |
| 6 | When does cross-gamma matter most? | Large moves, correlated stress, options |
| 7 | What is the VaR aggregation formula? | $\sqrt{\sum_i \sum_j VaR_i VaR_j \rho_{ij}}$ |
| 8 | When is total VaR < sum of standalone VaRs? | When correlations $\rho_{ij} < 1$ |
| 9 | What is component VaR? | Marginal contribution that sums to total VaR |
| 10 | What hedges FX delta? | FX spot, forwards, or FX swaps |
| 11 | What hedges domestic PV01? | Domestic swaps, futures, or bonds |
| 12 | What hedges basis DV01? | Cross-currency basis swaps |
| 13 | Why is multi-curve important for risk? | Discount and projection PV01 can differ in sign |
| 14 | What does "bump-and-rebuild" mean? | Bump market quotes, rebuild curves, reprice |
| 15 | What is basis DV01? | PV sensitivity to 1bp move in basis spread $e$ |
| 16 | If you pay "foreign + basis," is basis DV01 positive or negative? | Negative (worse when basis widens) |
| 17 | What is the forward FX formula? | $F_0 = S_0 \cdot P_f(0,T) / P_d(0,T)$ |
| 18 | What is Euler's theorem used for in risk? | Decomposing total VaR into components |
| 19 | Why can correlation breakdown hurt? | Diversification benefit disappears in stress |
| 20 | What causes secondary exposures when hedging? | Hedge instruments have their own risk profiles |
| 21 | What is a quanto? | Derivative where underlying is in one currency, payoff in another |
| 22 | What creates quanto adjustment? | Correlation between underlying and FX rate |
| 23 | What are the three main PCA factors for rates? | Level (parallel), slope (twist), curvature (butterfly) |
| 24 | How much variance does PC1 typically explain? | 80-90% (parallel shift) |
| 25 | What is a factor model for correlation? | $U_i = a_i F + \sqrt{1-a_i^2} Z_i$ where F is common factor |

---

## Mini Problem Set

**Q1.** Define $S$ as USD/EUR. If spot moves from 1.10 to 1.12, and your FX delta is +50mm EUR, what is your P&L in USD?

*Solution sketch:* $\Delta PV = \Delta S \times \Delta_{FX} = 0.02 \times 50\text{mm} = 1.0\text{mm USD}$

**Q2.** Derive the FX delta for a portfolio with deterministic foreign cashflows $\{C_j, T_j\}$.

*Solution sketch:* $\Delta_{FX} = \partial/\partial S_0 [S_0 \sum_j C_j P_f(0,T_j)] = \sum_j C_j P_f(0,T_j)$

**Q3.** Explain why an FX forward used to hedge FX delta introduces domestic PV01.

*Solution sketch:* Forward value = $N(K \cdot P_d - S_0 \cdot P_f)$. Taking derivative w.r.t. domestic curve shows sensitivity.

**Q4.** Two desks have VaRs of \$80mm and \$60mm with correlation 0.5. Calculate total VaR.

*Solution sketch:* $\sqrt{80^2 + 60^2 + 2(80)(60)(0.5)} = \sqrt{6400+3600+4800} = \sqrt{14800} = \$121.7\text{mm}$

**Q5.** For the cross-currency basis swap in Example H, verify that basis DV01 has the correct sign.

*Solution sketch:* Pay "EUR + basis" means higher $e$ increases payments, lowering PV. Thus $\partial V/\partial e < 0$ and basis DV01 < 0.

**Q6.** Explain cross-gamma conceptually. When would you expect large cross-gamma?

*Solution sketch:* Cross-gamma measures how FX delta changes with rates. Large for options, callable structures, or any position where payoff depends nonlinearly on multiple factors.

**Q7.** A hedge program neutralizes FX delta but finds residual foreign PV01. Explain why and what to do.

*Solution sketch:* The FX forward has its own rate sensitivity. Hedge residual foreign PV01 with foreign-currency swaps, which will create new FX delta requiring iteration.

**Q8.** What happens to VaR aggregation diversification benefit if correlations go to 1.0 in stress?

*Solution sketch:* Total VaR becomes the sum of standalone VaRs—no diversification benefit. This is why stress testing often assumes higher correlations.

**Q9.** Explain why PCA is useful for managing multi-currency interest rate risk.

*Solution sketch:* PCA reduces the dimensionality of rates risk. Instead of tracking sensitivities to many individual rates, you track exposures to 2-3 factors (level, slope, curvature) that explain ~98% of variance. For multi-currency portfolios, you apply PCA to each curve separately, then model cross-currency correlations at the factor level.

**Q10.** A trader holds a quanto derivative on a EUR asset with payoff in USD. The correlation between the asset and EUR/USD is +0.3. If this correlation increases to +0.5, what happens to the fair value (qualitatively)?

*Solution sketch:* Higher positive correlation means the quanto adjustment increases in magnitude (more negative adjustment to the forward). This typically reduces the fair value of the derivative because when the asset does well, the currency also strengthens, but you receive the payoff in USD which is now weaker. The exact magnitude depends on the volatilities and time to maturity.

---

## Source Map

### (A) Verified Facts — Directly Source-Backed

| Content | Source |
|---------|--------|
| FX forward formula $F_0 = S_0 e^{(r_d - r_f)T}$ and quote direction warning | Hull OFD |
| Delta as $\partial V / \partial X$ | Hull OFD |
| Multi-curve separation of discount and forward curves | Andersen & Piterbarg |
| Sensitivity via perturbations to funding/index/basis instruments | Andersen & Piterbarg |
| Cross-currency basis swap PV form and par basis definition | Andersen & Piterbarg |
| Interest rate risk "more difficult to manage" due to term structure complexity | Hull RM Ch 9 |
| Gamma definition as second partial derivative | Hull OFD Ch 19 |
| For delta-neutral portfolio: $\Delta\Pi \approx \Theta\Delta t + \frac{1}{2}\Gamma\Delta S^2$ | Hull OFD Ch 19 |
| VaR aggregation formula $\sqrt{\sum_i \sum_j VaR_i VaR_j \rho_{ij}}$ | Hull RM Ch 12 |
| Component VaR and Euler's theorem | Hull RM Ch 12 |
| Post-crisis shift from LIBOR to OIS for discounting | Hull RM Ch 9 |
| PCA for interest rates: "typical changes in term structure shape" | Tuckman Ch 13 |
| PCA explains ~98% variance with 3 factors | Tuckman Ch 13, Hull RM Ch 9 |
| Factor model for correlation $U_i = a_i F + \sqrt{1-a_i^2} Z_i$ | Hull RM Ch 11 |
| Quanto definition: "underlying measured in one currency, payoff in another" | Hull OFD Ch 5, 30 |
| Diff swaps as quanto example | Hull OFD Ch 7 |

### (B) Reasoned Inference — Derived in This Chapter

| Content | Derivation Logic |
|---------|------------------|
| Multi-currency PV decomposition $PV_d = \sum C_d P_d + S \sum C_f P_f$ | Standard discounting + FX conversion |
| FX01 conversion $= 0.01 \cdot S \cdot \Delta_{FX}$ | Algebra from delta definition |
| Forward creates rate PV01 | Differentiate $V_{\text{fwd}} = N(K P_d - S P_f)$ w.r.t. curves |
| Cross-gamma Taylor expansion | Extension of Hull's single-asset gamma to multi-factor |
| Iterative hedging necessity | Hedges create secondary exposures; must re-measure |
| Quanto adjustment formula form | General structure from Hull Ch 30; specific coefficients model-dependent |
| PCA application to multi-currency | Apply single-currency PCA separately, then correlate factors |

### (C) Flagged Uncertainties

| Topic | Note |
|-------|------|
| Cross-currency basis quoting/sign conventions | I'm not sure without currency pair, trade direction, and desk conventions |
| Collateral currency/CSA-driven discounting | I'm not sure without CSA and clearing details |
| Exact FX swap settlement conventions | I'm not sure without pair/market specification |
| Cross-gamma magnitude in specific products | I'm not sure without product-specific model details |
| Risk system bump conventions | I'm not sure without system specification |
| Exact quanto adjustment formulas | I'm not sure without specifying the underlying rate/asset model (Hull develops specific cases in Ch 30) |
| Factor model calibration in practice | I'm not sure about exact calibration procedures without institutional context |

---

*Last Updated: January 2026*
