# Chapter 46: Intrinsic Index Spread and Index Basis

---

## Introduction

The index trades at 85 bp, but the constituents imply 82 bp. Where's the 3 bp?

That wedge is the **index basis**. An index has a **quoted** market level, but you can also compute a **bottom-up** level by valuing it as a portfolio of its constituent single-name CDS. The bottom-up level is often called the **intrinsic** index spread (or intrinsic index value). The basis is the difference between the quoted and intrinsic levels under a stated sign convention.

Why this matters is practical, not philosophical. A hedge can be “spread-neutral” on paper and still generate P&L if the **basis** moves. Basis is therefore a distinct risk factor: it is what remains after you match constituent-level spread risk with an index proxy.

For risk, operations, and product control, basis is also a common explanation for “P&L breaks” between an index book and the associated constituent book. If one side is marked off the quoted index and the other is marked off single-name curves (or stale single-name quotes), the two will not necessarily move together.

Prerequisites: [Chapter 41 — CDS Indices — Mechanics, Coupons, Rolls](chapter_41_cds_indices_mechanics_coupons_rolls.md), [Chapter 42 — Bootstrapping a CDS Survival Curve from Market Quotes](chapter_42_bootstrapping_cds_survival_curve.md), [Chapter 43 — Risks in CDS and Hedging Strategies](chapter_43_cds_risks_hedging.md), [Chapter 45 — CDS Indices — Structure, Quoting, and Lifecycle](chapter_45_cds_indices_structure_quoting_lifecycle.md)  
Follow-on: [Chapter 47 — Hedging and Relative Value in CDS Indices](chapter_47_hedging_relative_value_cds_indices.md), [Chapter 48 — CDO / Tranche Products — What They Are (Product Map)](chapter_48_cdo_tranche_products_product_map.md), [Chapter 49 — Tranche Core Concepts — Expected Tranche Loss and Present Value](chapter_49_tranche_core_concepts_etl_pv.md), [Chapter 50 — Correlation and Tranche Pricing Frameworks](chapter_50_correlation_tranche_pricing_frameworks.md)

## Learning Objectives
- Compute an intrinsic index level from constituent CDS spreads and risky annuities (RPV01s).
- Translate a quoted index level (fixed coupon + upfront) into cash settlement amounts and PV.
- Define and interpret index basis with explicit sign and units.
- Compute and interpret basis P&L and define a “Basis01” with explicit bump object, bump size, units, and sign.
- Explain key basis drivers (documentation, liquidity/technicals, composition/roll) and why “arbitrage” is not free.
- Explain why tranche/index models often apply a portfolio swap adjustment (PSA) so intrinsic matches quoted.

This chapter develops the framework for understanding and measuring the index basis:

1. **Intrinsic spread calculation** — How to compute what the index "should" trade at from its constituents
2. **The RPV01-weighted average approximation** — Why simple averaging is wrong and what the correct weighting is
3. **Index basis definition and measurement** — The wedge between quoted and intrinsic levels
4. **Basis drivers** — Documentation (restructuring clauses), liquidity, composition, roll effects, and crisis dynamics
5. **Limits to basis arbitrage** — Why the basis persists despite apparent arbitrage opportunities
6. **Portfolio swap adjustment** — How practitioners force intrinsic to equal quoted for model consistency
7. **Risk and hedging implications** — Why the basis matters for P&L attribution and hedge design

Understanding the index basis connects directly to the mechanics covered in **Chapter 45** (CDS Index Structure), prepares you for the hedging strategies in **Chapter 47**, and is essential for the tranche pricing framework in **Chapters 48-50**, where model calibration requires that intrinsic equals quoted.

---

## Setup (Quotes, Day Count, Sign)

| Convention | Description |
|------------|-------------|
| **Instrument focus** | CDS portfolio indices (CDX / iTraxx) treated as equal-notional portfolios of single-name CDS unless explicitly stated |
| **Premium accrual** | Quarterly premium; ACT/360 |
| **Discounting** | $P(t,u)$ is the risk-free discount factor from $t$ to $u$ |
| **Credit modeling** | Each name $m$ has default time $\tau_m$, recovery $R_m$, and survival probability $Q_m(t,u) = \mathbb{P}(\tau_m \gt u \mid \mathcal{F}_t)$ |
| **Quote object** | Standardized fixed coupon $C(T)$ plus an upfront $U(t,T)$ at trade/settlement (unfunded index) |
| **Index basis sign** | $b(t,T) := S_{\text{quoted}}(t,T) - S_{\text{intrinsic}}(t,T)$ (positive basis = index quoted wider than intrinsic) |
| **1bp** | $1\text{bp}=10^{-4}$ in decimal spread units |

---

## 46.1 The Intrinsic Spread: What the Index "Should" Trade At

### 46.1.1 The Economic Intuition

A CDS index is economically a portfolio of single-name CDS contracts. If the index contract were identical to the replicating basket and you could trade the full basket without frictions, the index’s fair level would equal the basket-implied level. That basket-implied level is what we call the **intrinsic** index spread (or intrinsic index value).

The intrinsic spread answers a replication question: *if I replicated the index by trading the constituents, what flat index level would that portfolio imply under the market’s index-quoting convention?*

### 46.1.2 From Constituent PVs to Intrinsic Value

To derive the intrinsic spread, start from the present values of the protection and premium legs for each constituent, then use PV additivity for an equal-notional basket. Under standard simplifying assumptions (equal weights $1/M$, specified recoveries, independence between default and rates, etc.), it is convenient to express values in terms of each name’s par spread and its risky annuity (RPV01).

For the **protection leg**, a credit event on name $m$ results in a loss of $(1-R_m)/M$ to a long-protection index position. Using the single-name par identity “protection-leg PV = spread $\times$ RPV01,” the index protection-leg PV can be written as:

$$\text{Index protection leg PV}(t) = \frac{1}{M} \sum_{m=1}^{M} S_m(t,T) \cdot RPV01_m(t,T)$$

For the **premium leg**, contractually, a credit event reduces the spread payments by factor $1/M$. The premium leg value is:

$$\text{Index premium leg PV}(t) = \frac{C(T)}{M} \sum_{m=1}^{M} RPV01_m(t,T)$$

Therefore the intrinsic value can be written (in RPV01 form) as:

**Short protection (premium leg minus protection leg):**
$$\boxed{V_{\text{short}}(t,T) = \frac{1}{M} \sum_{m=1}^{M} \left( C(T) - S_m(t,T) \right) RPV01_m(t,T)}$$

**Long protection (protection leg minus premium leg):**
$$\boxed{V_{\text{intrinsic}}(t,T) := -V_{\text{short}}(t,T) = \frac{1}{M} \sum_{m=1}^{M} \left( S_m(t,T) - C(T) \right) RPV01_m(t,T)}$$

This formula has an elegant interpretation:
- $(S_m - C)$ is the "off-market" amount for name $m$—how much its par spread differs from the index coupon
- $RPV01_m$ is the risky annuity per unit notional (years); multiply by $N$ and $1\text{bp}=10^{-4}$ to get dollars per bp
- The product $(S_m - C)\times RPV01_m$ is the upfront contribution in points of notional (and in dollars after multiplying by $N$)

**Sanity checks:**
- If $S_m = C$ for all names, intrinsic value is zero (the index is "at par")
- If spreads widen (holding coupon fixed), long protection value increases: $V_{\text{intrinsic}} \uparrow$

### 46.1.3 The Risky PV01 (RPV01)

RPV01 is the key building block for CDS valuation. Under a continuous-premium approximation, it can be written as:

$$\boxed{RPV01_m(t,T) \equiv \int_t^T P(t,u) \, Q_m(t,u) \, du}$$

This represents the present value of receiving USD 1 per year, continuously, until default or maturity—whichever comes first. The survival probability $Q_m(t,u)$ ensures we only count premium periods where the name survives.

**Check (continuous vs discrete):** in production, $RPV01$ is computed on the actual premium schedule (quarterly dates, ACT/360 accrual) and includes the expected premium accrued if default happens between coupon dates. The continuous-time integral is a compact intuition anchor: it makes the dependence on discounting and survival explicit and makes the units obvious.

**Unit analysis:**
- $P(t,u)$ is unitless (discount factor)
- $Q_m(t,u)$ is unitless (probability)
- $du$ has units of years
- Therefore RPV01 has units of years—a "risky duration" of premium payments

For a name with higher default risk (lower survival probability), the RPV01 is lower because the expected premium-paying period is shorter. This has important implications for how we weight constituent spreads when computing intrinsic.

> **Desk Reality: How Traders Think About RPV01**
>
> “RPV01” is often used as shorthand for “risky duration.” A distressed name can have a much shorter expected premium-paying life than an investment-grade name at the same maturity, so it naturally receives less weight in an intrinsic calculation.

### 46.1.4 The Exact Intrinsic Spread Definition

The intrinsic spread $S_{\text{intrinsic}}$ is defined as the **flat** index spread that, when applied to the index under the market’s flat-curve quoting convention, produces the same upfront value as the bottom-up constituent calculation.

Mathematically, we calculate the upfront value of the index using a flat index curve as:

$$U_I(t) = (S_I(t,T) - C(T)) \cdot RPV01_I(t,T)$$

Equating the bottom-up intrinsic upfront and the flat-curve index upfront:

$$\boxed{\frac{1}{M} \sum_{m=1}^{M}\left(S_{m}(t, T)-C(T)\right) \cdot \mathrm{RPV01}_{m}(t, T)=\left(S_{\mathrm{intrinsic}}(t, T)-C(T)\right) \cdot \mathrm{RPV01}_{I}(t, T)}$$

Here $RPV01_I$ is calculated using a flat index curve—a market convention for index pricing. Because $RPV01_I$ itself depends on the spread level, this equation is mildly nonlinear and requires solving iteratively (e.g., via Newton-Raphson or bisection).

**Check (why “RPV01 fixed” can overstate spread-to-upfront mapping at high spreads):** if (schematically) the index upfront is $U(S)\approx (S-C)\,RPV01_I(S)$, then
$$
\frac{dU}{dS}=RPV01_I(S) + (S-C)\,\frac{d\,RPV01_I}{dS}.
$$
Higher spreads typically imply shorter survival and a **smaller** $RPV01_I$, so $d\,RPV01_I/dS\lt 0$. That means the constant-$RPV01$ approximation tends to **overstate** $\Delta U$ for a given $\Delta S$ when spreads are high.

### 46.1.5 The Operational Approximation: RPV01-Weighted Average

For practical purposes, a close approximation avoids solving the nonlinear equation. If the individual spread curves are reasonably homogeneous, we can approximate:

$$RPV01_I(t,T) \simeq \frac{1}{M} \sum_{m=1}^{M} RPV01_m(t,T)$$

This allows cancellation of the coupon term, yielding:

$$\boxed{S_{\text{intrinsic}}(t,T) \approx \frac{\sum_{m=1}^{M} S_m(t,T) \cdot RPV01_m(t,T)}{\sum_{m=1}^{M} RPV01_m(t,T)}}$$

**Derivation sketch:**
1. Start from the exact equation above
2. Approximate $RPV01_I \approx \frac{1}{M} \sum_m RPV01_m$
3. The coupon term cancels on both sides
4. Solve for $S_{\text{intrinsic}}$

**Why RPV01 weighting, not equal weighting?**

A simple average of constituent spreads would be wrong because it ignores the different "durations" of premium exposure across names. A name trading at 1000 bp has high default probability and short expected premium-paying life (low RPV01). Weighting by RPV01 reflects that we expect to pay spread to that name for less time.

### 46.1.6 The Concavity Effect: Why Dispersion Pulls Intrinsic Down

The key mechanical point is simple: **RPV01 tends to be smaller for wider (riskier) names**, because their expected premium-paying life is shorter. When you compute intrinsic as an RPV01-weighted average, you automatically give distressed names less weight than an equal-weight average would.

Write the intrinsic spread as a weighted average:

$$S_{\text{intrinsic}} \approx \sum_{m=1}^{M} w_m S_m, \qquad w_m := \frac{RPV01_m}{\sum_{k=1}^{M} RPV01_k}.$$

If (in your model and in the cross section of names you are looking at) $RPV01_m$ is decreasing in $S_m$, then high-spread names receive smaller weights and the intrinsic spread is pulled toward the tighter names.

**Numerical demonstration (2-name toy example, hypothetical):**

| Name | Spread (bp) | RPV01 (years) | Weight |
|------|-------------|---------------|--------|
| A (Safe) | 50 | 4.5 | 4.5/5.0 = 90% |
| B (Distressed) | 1000 | 0.5 | 0.5/5.0 = 10% |

- **Simple average:** $(50 + 1000)/2 = 525$ bp
- **Intrinsic (RPV01-weighted):** $0.90 \times 50 + 0.10 \times 1000 = 45 + 100 = 145$ bp

The distressed name contributes 10% weight despite being 50% of the portfolio by count. The intrinsic spread (145 bp) is dramatically below the simple average (525 bp).

> **Deep Dive: The High-Yield Trap (Why Simple Average Lies)**
>
> Imagine a 2-name index and assume:
> *   **Name A**: 50 bp (safe), RPV01 $=4.5$ years.
> *   **Name B**: 5,000 bp (distressed), RPV01 $=0.5$ years.
>
> *   **Simple Average**: $(50 + 5000)/2 = 2,525$ bp.
> *   **Reality**: you expect to pay/receive the 5,000 bp premium for far fewer years because the distressed name is likely to default earlier.
> *   **Intrinsic**: The index is dominated by the *survivor* (Name A). The intrinsic spread will be much closer to Name A than the simple average suggests.
> *   **Lesson**: Never use simple average for high-dispersion portfolios.

### 46.1.7 Intrinsic vs Simple Average: A Quick Diagnostic

A useful diagnostic is to compute both:
- the equal-weight average spread, $\bar{S} := \frac{1}{M}\sum_{m=1}^{M} S_m$, and
- the intrinsic spread, $S_{\text{intrinsic}} \approx \frac{\sum_m S_m\,RPV01_m}{\sum_m RPV01_m}$.

The gap $\bar{S} - S_{\text{intrinsic}}$ is a quick proxy for dispersion and for how much the basket is dominated by a few high-spread/low-RPV01 names.

> **Desk Reality: Intrinsic reconciliation checklist**
>
> **Common break:** Two systems disagree on intrinsic (and therefore basis) even when they use the same constituent spreads.
>
> **What to check:** (1) are you using the same constituent list (after defaults/roll)? (2) are you aligning documentation (e.g., restructuring) between index and constituents? (3) are your curves/recoveries/discount factors consistent? (4) are you using the same timestamped data snapshot?

---

## 46.2 The Index Basis: Definition and Measurement

### 46.2.1 Defining the Basis

The **index basis** is the difference between where the index actually trades and where it "should" trade based on constituents:

$$\boxed{b(t,T) = S_{\text{quoted}}(t,T) - S_{\text{intrinsic}}(t,T)}$$

With this sign convention:
- $b \gt 0$: Index quoted **wider** than intrinsic (index "cheap" vs constituents)
- $b \lt 0$: Index quoted **tighter** than intrinsic (index "rich" vs constituents)

**Sanity check:** If the index is quoted at 85 bp and the constituents imply 82 bp, then $b=+3$ bp (positive basis = quoted wider than intrinsic).

> **Analogy: The ETF Mechanism**
>
> Think of the Index as an **ETF** and the Constituents as the **Stock Basket**.
>
> *   **Intrinsic Spread = NAV**: The Net Asset Value of the basket. This is the math.
> *   **Quoted Spread = ETF Price**: Where buyers/sellers actually trade the ticker. This is the market.
> *   **Basis = Premium/Discount**: Just like an ETF can trade at a premium to NAV during a buying frenzy, the CDS Index can trade wider than intrinsic during a credit panic.
> *   **Arbitrage (soft, not free):** If the gap gets too big, traders may buy the index and sell the basket (or vice versa). But execution, funding/margin, and documentation mismatches mean convergence is risky and not instantaneous.

### 46.2.2 Quoting Conventions Matter

The basis is only meaningful if **quoted** and **intrinsic** are expressed in the **same quote object**.

Two common conventions in practice are:

- **Spread-quoted index (bp):** you quote a par spread $S_{\text{quoted}}$ for a standardized fixed coupon $C$, and translate the quote into an upfront amount using the index risky annuity (RPV01).
- **Price / points-upfront quoted index:** you quote an upfront amount directly (often presented as a “price per 100”), and any “spread-equivalent” number depends on the RPV01 convention used to convert.

Whichever quote object you start from, **convert to a common representation first** (spread or upfront), compute intrinsic in the same convention, then take the difference.

**Check (spread basis ↔ upfront basis):** for spread-quoted indices under the local $RPV01$-fixed approximation,
$$
\Delta PV \approx N \cdot RPV01_I \cdot \Delta b,
$$
and the corresponding **clean upfront** change as percent of notional is approximately
$$
\Delta U_{\mathrm{pct}} \approx 0.01 \times A_I \times \Delta b_{\text{bp}}.
$$
Example: if $A_I=4.2$ years and $\Delta b=+3$ bp, then $\Delta U_{\mathrm{pct}}\approx 0.01\times 4.2\times 3 = 0.126\%$ (0.126 points), i.e., about $USD 126k$ per $USD 100\text{mm}$ notional.

> **Pitfall — Spread quote vs points-upfront:** Confusing “bp” quotes with “price/points” quotes (and forgetting the fixed coupon).
> **Why it matters:** You can compute the wrong settlement cash and the wrong basis by an order of magnitude.
> **Quick check:** Write down $(C,\ S_{\text{quoted}},\ RPV01,\ N)$ and recompute the upfront in (i) points of notional and (ii) dollars.

### 46.2.3 Basis P&L Impact

> **Desk Reality: Why Basis Matters for P&L**
>
> On a USD 100mm long-protection index position with RPV01 of 4.2 years, a **+3 bp basis widening** moves PV by about:
>
> $$\Delta \text{PV} \approx 100,000,000 \times 3 \times 10^{-4} \times 4.2 = +USD 126,000$$
>
> This is pure basis P&L—distinct from parallel spread moves. A portfolio manager who hedges single-name exposure with an index can be perfectly CS01-neutral and still experience this P&L from basis volatility.
>
> For short protection, the sign flips.
>
> **For middle office:** When you see a P&L break between index and constituent books that are supposedly hedged, basis movement is often the culprit. The basis is a distinct risk factor that survives CS01-neutral hedging.

---

## 46.3 Drivers of the Index Basis

The basis reflects frictions and structural differences between the index contract and the replicating basket. Common drivers include documentation/contract mismatches, liquidity and supply–demand technicals, roll/composition effects, default history (index factor mechanics), and limits to arbitrage.

### 46.3.1 Documentation Differences: Restructuring Clauses

**Anchor:** A basis can arise from documentation differences; in the case of the North American CDX index, index protection is triggered only by bankruptcy or failure to pay (restructuring excluded; “No-Re”), while the market standard for US single-name CDS is based on the “Mod-Re” restructuring clause, and No-Re spreads are typically about $5\%$ lower than Mod-Re spreads—so intrinsic computed from Mod-Re single-name quotes can show an immediate basis.

**Expand:** Mechanically, if the index excludes a credit event that is included in the single-name quotes you use, the index provides less protection than your “replicating” basket, so it can trade tighter even when the underlying credit risk feels similar.

**Check (rule-of-thumb, not a law):** Suppose you compute intrinsic from restructuring-including single-name quotes and get 100 bp. If you believe the relevant index contract is effectively “No-Re-like” and you apply a $5\%$ mapping, then a crude doc-adjusted intrinsic would be 95 bp. The only point is directional: **documentation mismatch can easily be worth a few bp**.

Always treat this as a contract-level issue: the right adjustment depends on the exact definitions, deliverables, and how your market quotes are constructed.

### 46.3.2 Liquidity and Supply-Demand Technicals

Indices are often more liquid than single names: tighter bid/ask, easier to trade size, and frequently the first instrument used for macro hedging. That can create basis through two channels:

1. **Liquidity premium differential:** a more liquid instrument can trade “tighter” than a less liquid replicating basket (lower spread for the same expected loss).
2. **Price discovery timing:** in fast markets, the index can move first while constituent quotes lag (or are stale), so a mid-quote intrinsic can temporarily look “wrong” versus the quoted index.

In benign markets, the sign can flip depending on how intrinsic is measured (mid vs executable), documentation alignment, and microstructure.

**Bid-ask dispersion effect:** Even in calm markets, intrinsic computed from mid-market quotes may not be executable. If constituent bid-ask spreads are wide, the "executable intrinsic" (using bids or asks consistently) can differ from mid-intrinsic by several basis points.

### 46.3.3 Composition and Roll Effects

Many major index families roll on a schedule (often semi-annually), refreshing the constituent list. Rolling involves:
- Changes in composition (names dropped for downgrades, liquidity, or defaults; new names added)
- Maturity extension of approximately six months (if credit curves slope upward, this widens the new series relative to the old)

These effects can create apparent basis changes across series and between on-the-run and off-the-run indices:
- A new on-the-run series with fresh (higher-quality) names may trade tighter than an old series with deteriorated credits
- Comparing intrinsic across series requires using the appropriate constituent list for each

### 46.3.4 Default and Constituent Removal

When a name defaults:
- It is removed from the index without replacement
- The index notional reduces by $1/M$
- Future coupon payments shrink accordingly
- Accrued coupon at default is settled as part of premium leg mechanics

These mechanics mean that indices with different default histories have different effective portfolios. Computing intrinsic requires knowing exactly which names remain, the current index factor/outstanding notional, and the contract conventions your system assumes for accrued premium and default settlement.

### 46.3.5 Limits to Basis Arbitrage

The ETF analogy suggests that if the index trades at a premium to intrinsic, you should buy the index and sell the constituents (or vice versa), capturing the basis. In practice, several frictions prevent this:

**1. Execution friction:** Trading a large basket of single-name CDS simultaneously is operationally difficult. Bid-ask spreads, market impact, and execution timing all erode the apparent arbitrage profit.

**2. Capital constraints:** Basis trades require balance sheet. You must post margin on both the index and constituent legs, tying up capital. The return on capital may not justify the trade at small basis levels.

**3. Documentation mismatch creates real risk:** If the index and constituents do not reference identical contract terms (e.g., restructuring treatment), you do not have a pure arbitrage—you have contract-mismatch event risk.

**4. Timing risk (convergence uncertainty):** Basis may not converge before the index rolls. If you put on a basis trade expecting convergence, you may need to roll the position—at a cost—or close at a loss if the basis persists.

**5. Margin funding and carry:** Even though index and constituents are both CDS, the margin you post is funded. Carry and financing costs can dominate the economics of small basis trades.

> **Desk Reality:** Basis trades are sized and risk-managed as *convergence trades*, not as risk-free arbitrage.
> **Common break:** You size off mid intrinsic, ignore execution/margin, and are forced out by mark-to-market before convergence.
> **What to check:** Build an executable intrinsic band, stress basis widening, and verify contract/series/factor alignment on both legs.

### 46.3.6 Basis Dynamics in Stress Periods

**Stylized pattern (not a rule):**
- In sell-offs, the quoted index spread can move faster than computed intrinsic (especially if intrinsic is built from mid quotes in less-liquid names), producing basis moves.
- As liquidity returns and single-name markets re-open, basis can narrow, but convergence timing is uncertain.
- The sign and size of the basis are path-dependent: documentation mismatches (No-Re vs Mod-Re), bid-ask dispersion, and roll effects can dominate.

There is no universal “typical” stress-period basis magnitude. To quantify basis behavior for a specific index family and date range, you need a historical time series (quoted index, constituent quotes, and the exact intrinsic calculation rule) and then summarize the resulting distribution (levels, tails, and persistence).

> **Desk Reality: Basis During a Sell-Off**
>
> When you see basis blow out:
> 1. Don't assume it's a "signal" to buy index vs constituents—it may widen further
> 2. Recognize that your index hedges may underperform in the short term
> 3. P&L attribution should separate "spread P&L" from "basis P&L"
> 4. Liquidity in single-names may be severely impaired—don't assume you can execute the "arbitrage"

### 46.3.7 Cross-Series and Cross-Index Basis

**On-the-Run vs Off-the-Run Series Basis:**

After a roll, liquidity migrates to the new on-the-run series. The old (off-the-run) series continues to trade but with wider bid-ask spreads and less depth. This creates:

- **Liquidity-driven basis:** Off-the-run may trade at a discount (tighter) to its intrinsic as sellers accept worse prices for liquidity
- **Composition-driven basis:** If the old series has experienced defaults or downgrades not reflected in the new series, intrinsic levels differ
- **Hedging risk:** Using on-the-run index to hedge off-the-run positions introduces series basis risk in addition to standard basis

If you hedge a legacy/off-the-run exposure with the liquid on-the-run series, you are explicitly accepting series basis P&L as the cost of liquidity.

**CDX vs iTraxx Geographic Basis:**

Different index families can have different compositions, documentation choices (including credit-event/restructuring treatment), and market dynamics. Cross-index “basis” trades therefore embed multiple risks beyond a simple spread view.

Cross-index basis trades (long one, short the other) are occasionally implemented as macro bets on US vs European credit. However, these are not arbitrage—they are directional views on relative credit cycles with basis as the expression.

---

## 46.4 The Portfolio Swap Adjustment (PSA)

### 46.4.1 Why the Adjustment Is Needed

**Anchor:** If you want to risk-manage a CDS index with respect to its constituents, the index basis creates a mismatch: the basket-implied (intrinsic) index level from constituent curves can differ from the market-quoted index level.

One approach is the **portfolio swap adjustment (PSA)**: adjust the individual CDS spreads (issuer curves) so that the adjusted intrinsic index spread equals the market-quoted index spread (typically at one or more calibration maturities).

**Expand:** We choose to adjust issuer curves rather than the index swap since the index swap is substantially more liquid and its prices are treated as more certain than the CDS quotes. The exact nature of the adjustment is not unique, so in practice you prefer rules that are stable, fast, avoid arbitrage, preserve each issuer’s term-structure “shape,” and do not change relative risk ranking.

### 46.4.2 Principles of the Adjustment

Desirable properties for a PSA include:

1. **Proportional, not absolute:** prefer scaling spreads/hazards by a factor rather than adding the same bp to every name.
2. **Arbitrage-free:** do not induce negative hazards or non-monotone survival curves.
3. **Preserve term structure shape:** avoid warping each issuer curve in a way that creates unrealistic kinks.
4. **Preserve relative ranking:** do not reorder names’ relative riskiness.
5. **Fast:** PSA is often a preprocessing layer for tranche/option analytics, so it must be computationally cheap.

> **Why Proportional, Not Absolute?**
>
> Consider a 10% proportional adjustment applied to two names:
> - Name A (tight): 20 bp → 22 bp (2 bp increase = 10% of its risk)
> - Name B (wide): 200 bp → 220 bp (20 bp increase = 10% of its risk)
>
> Compare to an absolute 20 bp bump:
> - Name A: 20 bp → 40 bp (100% increase in risk—doubling!)
> - Name B: 200 bp → 220 bp (10% increase)
>
> The absolute bump would dramatically distort the relative risk profile, making tight names appear much riskier relative to wide names. Proportional adjustment preserves the "shape" of the credit portfolio.

### 46.4.3 The Spread Multiplier Approach

One approach multiplies all constituent spreads by a common factor $\alpha(T)$:

$$S_{m}^{*}(t, T)=\alpha(T) \cdot S_{m}(t, T)$$

At each calibration maturity, solve for $\alpha(T)$ so that the intrinsic computed from the adjusted constituents matches the quoted index.

**Algorithm sketch:**

1. Build all $M$ survival curves from original spreads
2. Calculate interpolated CDS spreads to index maturity dates
3. Initialize $\alpha = 1$
4. Apply $\alpha$ to spreads, rebuild curves, compute intrinsic
5. Update $\alpha$ based on how far intrinsic is from quoted
6. Iterate until convergence
7. Stop when the quoted vs intrinsic mismatch is within tolerance (or when you hit a maximum iteration count)

### 46.4.4 The Survival Probability Multiplier (Faster)

A more efficient approach adjusts survival probabilities directly, avoiding full curve rebuilding at each iteration:

Define the **forward** survival probability over $[T_{n-1},T_n]$ as
$$Q_m(t;T_{n-1},T_n):=\frac{Q_m(t,T_n)}{Q_m(t,T_{n-1})}.$$

Then the PSA idea is to raise that forward survival probability to a power:
$$Q_{m}^{*}\left(t, T_{n}\right)=Q_{m}^{*}\left(t, T_{n-1}\right)\cdot \bigl(Q_m(t;T_{n-1},T_n)\bigr)^{\alpha(n)}.$$

This raises the forward survival probability over $[T_{n-1},T_n]$ to a power $\alpha(n)$. Under an intensity representation, it is equivalent to multiplying the forward hazard rate over that interval by $\alpha(n)$.

This can be faster because it avoids bootstrapping survival curves inside the iteration.

### 46.4.5 Worked Example: PSA in Practice

**Toy example:** At 5Y, suppose the quoted index level is 24 bp while the unadjusted intrinsic computed from constituents is 27 bp. A simple proportional PSA uses the ratio

$$\alpha = \frac{24}{27} \approx 0.889.$$

You then apply $\alpha$ (either to spreads or to forward hazard rates, depending on your implementation), recompute intrinsic, and check that the basket-implied index level matches 24 bp (within tolerance). See Example 6 for a fully worked toy calibration.

### 46.4.6 What Happens If You Skip PSA

> **Desk Reality: The Tranche Calibration Failure Mode**
>
> If you price a tranche using raw constituent curves (where intrinsic ≠ quoted), your model has a built-in PV difference vs the index. Here's what goes wrong:
>
> **Setup:** You're pricing a 3-7% CDX IG tranche. The index trades at 60 bp quoted, but your constituent curves imply 65 bp intrinsic (basis = -5 bp).
>
> **Problem:** Your tranche model uses constituent curves to build a loss distribution. The expected loss and protection leg PV are computed as if the index trades at 65 bp (your intrinsic). But when you delta-hedge using the quoted index, you're hedging a 60 bp instrument.
>
> **Result:** Systematic P&L breaks equal to the basis times your position size. Every day, your model says one thing, the market another.
>
> **Solution:** Apply PSA to scale constituent curves so intrinsic = 60 bp. Now your tranche model and your index hedge are calibrated to the same underlying.
>
> **Numerical example:** On a USD 50mm tranche with index RPV01 of 4.0 years and 5 bp basis:
> $$\text{Daily PnL break risk} \approx 50{,}000{,}000 \times 5 \times 10^{-4} \times 4.0 = USD 100{,}000$$
>
> This isn't a one-time error—it's a systematic mismatch that compounds over time.

---

## 46.5 Risk Measurement and Hedging Implications

### 46.5.1 Basis01: Sensitivity to Basis

**Anchor (definition + bump):** Define **Basis01** as the PV change for a **+1bp** widening of the basis
$$b(t,T)=S_{\text{quoted}}(t,T)-S_{\text{intrinsic}}(t,T),$$
implemented as: bump the quoted index level $S_{\text{quoted}}$ up by $+1\text{bp}=10^{-4}$ while holding the constituent snapshot (and therefore $S_{\text{intrinsic}}$) fixed.

- **Bump object:** quoted index par spread $S_{\text{quoted}}$
- **Bump size:** $+1\text{bp}=10^{-4}$
- **Units:** currency per 1bp for the stated notional
- **Sign convention (this chapter):** long protection has **positive** Basis01 (basis widening helps long-protection index PV)

**Expand:** To first order (ignoring the small change in RPV01 under the bump),
$$\Delta PV \approx N \cdot RPV01_I(t,T)\cdot \Delta b,$$
so
$$\boxed{Basis01 \approx N \cdot RPV01_I(t,T)\cdot 10^{-4}.}$$

**Check (numbers):** For $N=USD 100\text{mm}$ and $RPV01_I=4.0$ years, $Basis01\approx USD 40{,}000/\text{bp}$. A +3bp basis widening is roughly +USD 120k for a long-protection index position.

### 46.5.2 P&L Decomposition: Index vs Constituents

Consider a book with both index and constituent positions. Total P&L decomposes as:

$$\Delta \text{PV} \approx \underbrace{N_I \cdot CS01_I \cdot \Delta S_{\text{quoted}}}_{\text{Index spread PnL}} - \underbrace{\sum_m N_m \cdot CS01_m \cdot \Delta S_m}_{\text{Constituent PnL}}$$

Here `CS01` is the PV change for a **+1bp** bump (i.e., $+10^{-4}$ in decimal spread units) to the relevant par-spread quote (index or single-name), using your chosen curve-rebuild rule; units are currency per 1bp for the stated notional. In this chapter, long protection has positive CS01.

If you've hedged to be CS01-neutral ($N_I \cdot CS01_I = \sum_m N_m \cdot CS01_m$), parallel moves cancel. But residual P&L emerges from:
- **Basis moves:** $\Delta S_{\text{quoted}} \neq \sum_m w_m \Delta S_m$
- **Idiosyncratic moves:** Individual names move differently than the weighted average
- **Convexity:** Large moves where first-order hedging breaks down

### 46.5.3 Hedging an Index with Constituents

To hedge a long-protection index position with short-protection constituent trades, match CS01:

$$\text{Index CS01} = N_I \times 10^{-4} \times RPV01_I$$

$$\text{Constituent CS01} = \sum_m N_m \times 10^{-4} \times RPV01_m$$

For equal constituent notionals, set:
$$N_m = \frac{N_I \cdot RPV01_I}{\sum_m RPV01_m}$$

**Residual risks (failure modes):**
- Basis changes (quoted index moves without corresponding constituent move)
- Documentation mismatch (restructuring clause differences)
- Liquidity mismatch (constituents harder to trade at scale)
- Default mechanics (after default, notional and premium base change)

### 46.5.4 Hedging Constituents with an Index (Proxy Hedge)

When single-name positions are illiquid, the index provides a liquid proxy hedge. This works best when:
- The book is "index-like" (similar composition)
- Basis is stable
- Idiosyncratic risk is accepted

In stressed markets, this hedge can break down exactly when you need it most: the liquid index can move first while constituent quotes lag, and basis can move sharply.

### 46.5.5 Systematic vs Idiosyncratic Delta

**Chapter 47** develops this framework in detail. The key insight is that spread risk decomposes into:

- **Systematic (index) delta:** Exposure to parallel moves in the index level
- **Idiosyncratic delta:** Exposure to individual name moves that don't affect the index average

For a pure index position, idiosyncratic delta is zero (by construction—you hold all names in weight). For a constituent position, both are nonzero. Basis risk emerges when systematic deltas are matched but basis itself moves.

> **Desk Reality: When to Accept Basis Risk**
>
> **Macro hedging:** Basis risk is often acceptable. You want systematic credit exposure hedged; if your IG portfolio is highly correlated with an IG index, the index hedge may be “good enough.” Accept that some basis P&L is the cost of using a liquid proxy.
>
> **Basis trading:** You're explicitly betting on basis convergence. Size the trade based on how much basis P&L you can stomach if it moves against you before converging.
>
> **Model calibration (tranches):** You must eliminate basis via PSA. Tranche Greeks depend on a consistent model; a 5 bp basis mismatch can distort delta hedges significantly.
>
> **P&L attribution:** When explaining to your P&L committee that you're CS01-flat but made/lost money, basis is the answer. A good risk report separates "spread P&L" from "basis P&L" to make this transparent.

---

## 46.6 Worked Examples

### Example 1: Intrinsic Spread Calculation (5-Name Toy Index)

**Inputs:**
| Name | Spread $S_m$ (bp) | RPV01 (years) |
|------|-------------------|---------------|
| 1 | 50 | 4.4 |
| 2 | 60 | 4.2 |
| 3 | 70 | 4.0 |
| 4 | 80 | 3.8 |
| 5 | 90 | 3.6 |

**Step 1: Compute weighted numerator**
| Name | $S_m \times RPV01_m$ |
|------|---------------------------|
| 1 | $50 \times 4.4 = 220$ |
| 2 | $60 \times 4.2 = 252$ |
| 3 | $70 \times 4.0 = 280$ |
| 4 | $80 \times 3.8 = 304$ |
| 5 | $90 \times 3.6 = 324$ |
| **Sum** | **1380** |

**Step 2: Compute denominator**
$$\sum_m RPV01_m = 4.4 + 4.2 + 4.0 + 3.8 + 3.6 = 20.0$$

**Step 3: Intrinsic spread**
$$S_{\text{intrinsic}} = \frac{1380}{20.0} = 69.0 \text{ bp}$$

**Comparison with simple average:**
$$\bar{S} = \frac{50 + 60 + 70 + 80 + 90}{5} = 70.0 \text{ bp}$$

The intrinsic is 1 bp lower because higher-spread names have lower RPV01 (shorter expected premium life) and thus receive less weight.

---

### Example 2: Basis Calculation and Sign Interpretation

**Given:**
- From Example 1: $S_{\text{intrinsic}} = 69.0$ bp
- Market quoted index spread: $S_{\text{quoted}} = 72.0$ bp

**Basis:**
$$b = 72.0 - 69.0 = +3.0 \text{ bp}$$

**Interpretation:** The positive basis means the index is trading **wider** than its constituent-implied level. Possible explanations:
- Heavy protection buying in the index (hedging demand)
- Constituent quotes stale or not reflecting latest information
- Documentation or liquidity premium embedded in index

---

### Example 3: Attributing Basis to Drivers (Decomposition)

**Scenario:** CDX NA IG index trades at 70 bp. Using Mod-Re single-name spreads, intrinsic is 75 bp (basis = −5 bp). Investigate.

**Analysis:**

1. **Documentation adjustment:** Suppose the index contract excludes restructuring while your constituent quotes include it. As a rough mapping, assume the index-equivalent spread is $0.95\times$ the restructuring-including spread.
   - Adjusted intrinsic: $75 \times 0.95 = 71.25$ bp
   - Documentation-adjusted basis: $70 - 71.25 = -1.25$ bp

2. **Liquidity premium:** Index more liquid → lower spread. Estimate 0.5 bp liquidity advantage.
   - Residual: $-1.25 + 0.5 = -0.75$ bp

3. **Measurement error / bid-ask:** Within bid/ask and data-snapshot noise

**Decomposition summary:**
| Driver | Contribution (bp) |
|--------|-------------------|
| Restructuring (No-Re vs Mod-Re) | -3.75 |
| Liquidity premium | +0.5 |
| Residual / noise | -0.75 |
| **Total basis** | **-5.0** |

**Conclusion:** Most of the apparent −5 bp basis is explained by the documentation mismatch (−3.75 bp from restructuring). After adjustment, the index trades only slightly tight to intrinsic.

---

### Example 4 (Worked): From Quote to Settlement Cash (Upfront + Accrued)

**Context**
- You trade a spread-quoted CDS index with a fixed coupon $C$ and a quoted par spread $S_{\text{quoted}}$.
- You want to translate the quote into (i) upfront points, (ii) dollars, and (iii) settlement cash including accrued premium.

**Timeline (Concrete Dates)**
- Trade date: 2026-01-15
- Settlement date: 2026-01-16 (assumption for this example)
- Current accrual period: 2025-12-20 to 2026-03-20
- Next coupon payment date: 2026-03-20

**Inputs**
- Notional: $N=USD 100{,}000{,}000$
- Index coupon: $C=60$ bp
- Quoted index level: $S_{\text{quoted}}=72$ bp
- Index RPV01: $RPV01_I=4.5$ years (per unit notional, from your index pricer at the quoted level)
- Day count: ACT/360

**Outputs (What You Produce)**
- Upfront $U$ in points (fraction of notional) and dollars
- Accrued premium to settlement in dollars
- Settlement cash amount (upfront + accrued)
- Basis/Basis01 intuition check

**Step-by-step**
1. **Translate quote to upfront points**
   - Spread difference: $\Delta S := S_{\text{quoted}}-C = 12\text{ bp} = 12\times 10^{-4}=0.0012$
   - Upfront (points of notional): $U = \Delta S \cdot RPV01_I = 0.0012 \times 4.5 = 0.0054$
   - Interpretation: $U=0.54\%$ of notional (54 points per 10,000 notional, or “54 bp points”)

2. **Convert upfront points to dollars**
   $$U_{\mathrm{USD}} = N \cdot U = 100{,}000{,}000 \times 0.0054 = USD 540{,}000.$$

3. **Compute accrued premium to settlement**
   - Accrual fraction to settlement: $\tau=\frac{27}{360}=0.075$ (27 actual days from 2025-12-20 to 2026-01-16)
   - Full quarter accrual fraction: $\tau_{\text{qtr}}=\frac{90}{360}=0.25$ (2025-12-20 to 2026-03-20)
   - Accrued running premium (dollars):
   $$Accrued_{\mathrm{USD}}=N \cdot C \cdot \tau = 100{,}000{,}000 \times 0.0060 \times 0.075 = USD 45{,}000.$$

4. **Settlement cash amount (buyer of protection)**
   $$\text{Settlement cash paid} = U_{\mathrm{USD}} + Accrued_{\mathrm{USD}} = 540{,}000 + 45{,}000 = USD 585{,}000.$$

**Cashflows (protection buyer sign convention: pay = negative)**
| Date | Cashflow | Explanation |
|---|---:|---|
| 2026-01-16 | $-USD 585{,}000$ | Upfront + accrued premium paid at settlement |
| 2026-03-20 | $-USD 150{,}000$ | Full quarter running coupon: $-N\cdot C\cdot \tau_{\text{qtr}}=-100\text{mm}\cdot 0.0060\cdot 0.25$ |

**P&L / Risk Interpretation**
- The upfront sign is intuitive: if $S_{\text{quoted}}\gt C$, the protection buyer pays upfront; if $S_{\text{quoted}}\lt C$, the protection buyer receives upfront.
- A quick “risk scalar” is $Basis01\approx N\cdot RPV01_I\cdot 10^{-4}=100{,}000{,}000\times 4.5\times 10^{-4}=USD 45{,}000/\text{bp}$ (buyer of protection).

**Sanity Checks**
- **Limit check:** If $S_{\text{quoted}}=C$, then $U=0$ and settlement cash is just accrued premium.
- **Units check:** spreads are per year; RPV01 is in years; upfront points are unitless; dollars = notional × points.
- **Sign check:** widening $S_{\text{quoted}}$ increases $U$ and increases long-protection PV.

**Debug Checklist (When Your Result Looks Wrong)**
- Did you convert bp to decimal (1bp = $10^{-4}$)?
- Is your RPV01 per unit notional (years), or already scaled to dollars per bp?
- Are you using the correct accrual dates and day count (ACT/360 here)?
- Are you mixing “upfront points” with “price per 100” conventions?

---

### Example 5: Bid-Ask Effect on Intrinsic

**Inputs:** Same 5-name index, but with bid/ask spreads:

| Name | Bid (bp) | Ask (bp) | RPV01 |
|------|----------|----------|-------|
| 1 | 49 | 51 | 4.4 |
| 2 | 58 | 62 | 4.2 |
| 3 | 68 | 72 | 4.0 |
| 4 | 78 | 82 | 3.8 |
| 5 | 88 | 92 | 3.6 |

**Intrinsic at bid:**
- Numerator: $49(4.4) + 58(4.2) + 68(4.0) + 78(3.8) + 88(3.6) = 1344.4$
- Intrinsic: $1344.4 / 20.0 = 67.22$ bp

**Intrinsic at ask:**
- Numerator: $51(4.4) + 62(4.2) + 72(4.0) + 82(3.8) + 92(3.6) = 1415.6$
- Intrinsic: $1415.6 / 20.0 = 70.78$ bp

**Result:** The executable intrinsic band is [67.22, 70.78] bp, a width of 3.56 bp. Even before "true" basis, microstructure creates significant uncertainty in fair value.

---

### Example 6: PSA Calibration (Toy)

**Setup:**
- 3-name index, quoted at 50 bp
- Constituent spreads: [40, 50, 70] bp
- All RPV01 = 4.0 years (equal for simplicity)

**Step 1: Unadjusted intrinsic**
$$S_{\text{intrinsic}} = \frac{40(4) + 50(4) + 70(4)}{12} = \frac{640}{12} = 53.33 \text{ bp}$$

**Step 2: Required adjustment**
Need intrinsic = 50 bp. Using spread multiplier $\alpha$:
$$\frac{\alpha(40 + 50 + 70) \times 4}{12} = 50$$
$$\alpha \times 53.33 = 50$$
$$\alpha = 0.9375$$

**Step 3: Adjusted spreads**
| Name | Original (bp) | Adjusted (bp) |
|------|---------------|---------------|
| 1 | 40 | 37.5 |
| 2 | 50 | 46.9 |
| 3 | 70 | 65.6 |

**Verification:** $(37.5 + 46.9 + 65.6)/3 = 50.0$ bp ✓

The proportional adjustment preserves relative ranking while forcing intrinsic to match quoted.

---

### Example 7: CS01-Neutral Hedge Design

**Position:** Long protection on USD 100mm index with RPV01 = 4.1 years

**Index CS01:**
$$CS01_I = USD 100mm \times 10^{-4} \times 4.1 = USD 41{,}000/\text{bp}$$

**Hedge:** Short protection on 5 constituents with RPV01 = [4.4, 4.2, 4.0, 3.8, 3.6] years

For equal notional per name $N_m$:
$$5 \cdot N_m \times 10^{-4} \times \frac{20}{5} = 41{,}000$$
$$N_m \times 10^{-4} \times 4.0 = 8{,}200$$
$$N_m = USD 20.5mm$$

**Hedge:** USD 20.5mm short protection per name (5 names, total $102.5mm notional) vs $100mm long protection index.

---

### Example 8: Basis Trade P&L Scenarios

**Position:** CS01-neutral (Example 7)
- Long protection index: CS01 = +USD 41k/bp
- Short protection constituents: CS01 = -USD 41k/bp

**Scenario (i): Parallel widening, no basis change**
- Index widens 20 bp, each name widens 20 bp
- Index P&L: $+20 \times 41k = +USD 820k$
- Hedge P&L: $-20 \times 41k = -USD 820k$
- **Net: $0$** (systematic risk hedged)

**Scenario (ii): Pure basis move**
- Constituents unchanged, index tightens 10 bp
- Index P&L: $-10 \times 41k = -USD 410k$
- Hedge P&L: $0$
- **Net: -USD 410k** (basis P&L)

**Scenario (iii): Idiosyncratic constituent widening**
- Index unchanged, Name 5 widens 100 bp (others flat)
- Index P&L: $0$
- Hedge P&L on Name 5: $-100 \times (20.5mm \times 10^{-4} \times 3.6) = -USD 738k$
- **Net: -USD 738k** (idiosyncratic risk)

---

### Example 9: Recovery Rate Sensitivity in Intrinsic Calculation

**Setup:** 3-name index with heterogeneous recovery assumptions

| Name | Spread (bp) | Recovery | Implied RPV01 (years) |
|------|-------------|----------|----------------------|
| A | 50 | 40% | 4.4 |
| B | 100 | 40% | 4.0 |
| C | 200 | 25% | 3.2 |

**Note:** Name C has lower recovery assumption (subordinated debt), which implies higher loss-given-default and therefore shorter RPV01 at the same spread. The RPV01 is computed using the standard formula; lower recovery increases implied hazard rate for a given spread, reducing RPV01.

**Intrinsic (heterogeneous recovery):**
$$S_{\text{intrinsic}} = \frac{50(4.4) + 100(4.0) + 200(3.2)}{4.4 + 4.0 + 3.2} = \frac{220 + 400 + 640}{11.6} = \frac{1260}{11.6} = 108.6 \text{ bp}$$

**Comparison: If all names used 40% recovery:**
- Name C's RPV01 at 40% recovery would be higher, say 3.6 years
- New denominator: $4.4 + 4.0 + 3.6 = 12.0$
- New intrinsic: $(220 + 400 + 200 \times 3.6)/12.0 = (220 + 400 + 720)/12.0 = 111.7$ bp

**Result:** Using accurate name-specific recoveries (with Name C at 25%) produces intrinsic of 108.6 bp versus 111.7 bp with flat 40%. The 3 bp difference arises because the low-recovery name has shorter risk duration and gets less weight.

> **Practitioner Note:** Most production systems default to 40% flat recovery for simplicity. The error is usually small for IG indices but can matter for HY or when specific names have known subordinated exposure.

---

### Example 10: Cross-Series Hedging Residual Risk

**Situation:** You hold USD 50mm long protection on an off-the-run IG index series (“Series $N$”). You want to hedge using the liquid on-the-run series (“Series $N\!+\!1$”).

**Key differences between series:**
- Series $N$ and $N\!+\!1$ have different constituents (names are added/removed at roll)
- Series $N\!+\!1$ has slightly longer remaining maturity (curve slope matters)
- Liquidity: on-the-run bid/ask is typically tighter; off-the-run can be materially wider

**Hedge design:**
- Match CS01: Both series have RPV01 ≈ 4.1 years
- Hedge notional: USD 50mm Series $N\!+\!1$ short protection (proxy hedge)

**Residual risks:**

1. **Composition mismatch:** Names present in Series $N$ but not in $N\!+\!1$ create idiosyncratic exposure. If one of those names widens 50 bp, your Series $N$ position gains, but your Series $N\!+\!1$ hedge does not respond.
   - Estimated impact (if $M=125$ names): $\approx USD 50\text{mm} \times (1/125) \times 50\text{bp} \times 4.1 \times 10^{-4} \approx USD 8{,}200$

2. **Series basis:** If Series $N$ trades at a different basis to its intrinsic than Series $N\!+\!1$, you have a basis mismatch.
   - If Series $N$ basis is -2 bp and Series $N\!+\!1$ is 0 bp, you're effectively short 2 bp of series basis.

3. **Maturity mismatch:** Series $N\!+\!1$ has more remaining maturity. If curves steepen (long end widens more), the hedge underperforms.

**P&L scenario:** Series $N$ and $N\!+\!1$ both widen 10 bp, but Series $N$ basis widens 3 bp while Series $N\!+\!1$ stays flat.
- Series $N$ P&L: +10 bp × USD 41k/bp = +$410k (spread) + 3 bp × $41k/bp = +$123k (basis) = +$533k
- Series $N\!+\!1$ P&L: -10 bp × $41k/bp = -$410k
- **Net: +USD 123k** (series basis slippage benefited you this time)

---

## 46.7 Practical Notes

### 46.7.1 Production Checklist

Before computing intrinsic and basis:

- **Index identity:** Series number, maturity, on-the-run vs off-the-run status
- **Constituent list:** Confirm which names are included (post any defaults)
- **Constituent spreads:** Consistent tenor, same valuation date, appropriate restructuring clause
- **Recovery assumptions:** State your recovery convention/policy (e.g., flat $R=40\%$ for senior unsecured) and apply it consistently
- **Discount curve:** Consistent across index and constituent pricing
- **Quoting convention:** Spread vs price/points-upfront; convert to common units before computing basis

### 46.7.2 Data Sourcing Considerations

> **Desk Reality:** Intrinsic is only as good as the constituent snapshot you feed it.
> **Common break:** The index is marked off a timestamped index print, while constituents are from a different time (or are stale/indicative), creating a “basis” that is really a data mismatch.
> **What to check:** Align timestamps (index level, constituent quotes, index factor) and compute an executable intrinsic band using bid/ask.

Practical data notes:
- Record whether constituent inputs are mid, bid, ask, or composite; intrinsic can shift by bp when you change this choice.
- Flag constituents with missing/indicative-only quotes; decide a policy (carry-forward, conservative side, exclude) and keep it consistent.
- Post-trade prints can be lagged and may not be directly comparable to executable quotes; treat them as a different data type.

### 46.7.3 Common Pitfalls

1. **Documentation mismatch:** Using Mod-Re constituent spreads without adjusting for No-Re index (CDX)
2. **Stale quotes:** Computing intrinsic from end-of-day constituent spreads vs intraday index moves
3. **Bid-ask conflation:** Using mid-market intrinsic for execution analysis
4. **Recovery inconsistency:** Different recovery assumptions across names or vs index
5. **Series confusion:** Comparing on-the-run index to off-the-run constituents with different composition
6. **Ignoring defaults:** Not adjusting for names that have already defaulted and been removed

### 46.7.4 Verification Tests

- **Weight check:** Equal weights sum to 1, or stated weights verified
- **Intrinsic stability:** Small spread bumps should produce small intrinsic changes
- **Sign convention:** Verify $b = S_{\text{quoted}} - S_{\text{intrinsic}}$ used consistently
- **Repricing test:** Plugging $S_{\text{intrinsic}}$ back into index PV formula should recover constituent-implied PV (within approximation error)

---

## Summary

1. A CDS index can be valued **bottom-up** as a portfolio of constituent CDS; this implies an **intrinsic** index level under a stated convention.
2. Intrinsic is not a simple average spread: an RPV01-weighted aggregation is the natural approximation because names have different risky annuities.
3. Dispersion matters: wider names tend to have smaller RPV01, so intrinsic is often below the simple average when a few names are distressed.
4. The **index basis** is $b=S_{\text{quoted}}-S_{\text{intrinsic}}$ under a stated sign convention (positive basis = quoted wider than intrinsic).
5. Basis drivers include documentation/contract mismatches, liquidity/technicals, roll/composition effects, and measurement/executability (bid/ask).
6. Basis is a distinct risk factor: a portfolio can be CS01-neutral and still generate P&L when basis moves.
7. “Arbitraging” basis is not free: execution, margin, contract mismatch risk, and convergence uncertainty limit trades.
8. A **portfolio swap adjustment (PSA)** adjusts constituent curves so intrinsic matches quoted; this is useful when pricing/calibrating products that depend on both.
9. Define **Basis01** explicitly (bump object/size/units/sign) to measure basis exposure; for long protection, $Basis01\approx N\cdot RPV01_I\cdot 10^{-4}$.
10. For tranche/index option calibration (Chapters 48–50), consistency (intrinsic = quoted) is a prerequisite before interpreting tranche Greeks or hedge ratios.

## Key Concepts

| Concept | Definition | Why It Matters |
|---|---|---|
| **Intrinsic level** | Bottom-up index level implied by valuing the constituent basket under the index quote convention | Replication benchmark; basis starts here |
| **Quoted level** | Market index quote (spread or points-upfront) for a standardized fixed coupon | What you can trade and hedge |
| **Index basis** | $b=S_{\text{quoted}}-S_{\text{intrinsic}}$ (this chapter’s sign convention) | Residual risk after “spread-neutral” hedging |
| **RPV01** | Risky annuity per unit notional (years): PV of 1 unit of running spread per year until default/maturity | Correct weighting for intrinsic and scaling for CS01/Basis01 |
| **Executable intrinsic** | Intrinsic computed from consistent bid/ask inputs (not just mids) | Prevents “fake basis” from non-tradeable marks |
| **PSA** | Adjustment to constituent curves so basket-implied index matches the quoted index | Needed for tranche/index-option consistency |
| **Basis01** | PV change for a +1bp increase in $b$ (with a stated bump implementation) | Makes basis exposure reportable and hedgeable |

## Notation

| Symbol | Meaning | Units / notes |
|---|---|---|
| $M$ | Number of constituents | count |
| $N$ | Notional | currency (e.g., USD) |
| $S_m(t,T)$ | Par spread of name $m$ | bp or decimal per year |
| $S_{\text{quoted}}(t,T)$ | Quoted index level | usually bp (spread quote) under a fixed coupon |
| $S_{\text{intrinsic}}(t,T)$ | Intrinsic index spread | bp (spread-equivalent) |
| $b(t,T)$ | Index basis | bp; $b=S_{\text{quoted}}-S_{\text{intrinsic}}$ |
| $C$ | Fixed coupon of the index series | bp per year |
| $U$ | Upfront (points of notional) | unitless fraction; dollars = $N\cdot U$ |
| $P(t,u)$ | Discount factor | unitless |
| $Q_m(t,u)$ | Survival probability of name $m$ | unitless |
| $RPV01_m,\ RPV01_I$ | Risky annuities (name/index) | years per unit notional |

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is the intrinsic index spread? | The spread implied by valuing the index bottom-up from constituent CDS curves under the index quote convention |
| 2 | What is the intrinsic spread approximation? | $S_{\text{intrinsic}} \approx \frac{\sum_m S_m\,RPV01_m}{\sum_m RPV01_m}$ |
| 3 | Why isn't intrinsic the simple average of constituent spreads? | Wider/riskier names typically have smaller RPV01 (shorter expected premium-paying life), so they receive less weight |
| 4 | Define index basis (sign convention) | $b = S_{\text{quoted}} - S_{\text{intrinsic}}$; positive basis means index quoted wider than intrinsic |
| 5 | Name three drivers of index basis | Documentation mismatch, liquidity/technicals, roll/composition effects (plus measurement/executability effects) |
| 6 | How does a restructuring-clause mismatch affect basis? | If the index provides less protection than the single-name quotes you use, it tends to trade tighter; align documentation or adjust intrinsic |
| 7 | What is RPV01 conceptually? | Risky annuity per unit notional (years): PV of paying 1 unit of running spread per year until default or maturity |
| 8 | Why can the index move before single names? | The index is often more liquid; single-name quotes can lag, so an intrinsic built from a stale snapshot can diverge |
| 9 | What is the portfolio swap adjustment (PSA)? | A calibration step that adjusts constituent curves so the basket-implied index matches the quoted index |
| 10 | Why is PSA useful for tranche/index option work? | You want one consistent set of curves that reproduces the quoted index before calibrating products built on the index |
| 11 | What happens to index notional after a default (equal-weight index)? | It is reduced by $1/M$ (via the index factor); future premium cashflows shrink accordingly |
| 12 | What's the spread-to-upfront mapping (spread-quoted index)? | Upfront points per unit notional: $U \approx (S_{\text{quoted}}-C)\cdot RPV01_I$ |
| 13 | If all constituent spreads and RPV01s are equal, what is intrinsic? | The common spread (weighted average reduces to simple average) |
| 14 | How does bid-ask dispersion affect intrinsic? | It creates an executable intrinsic *band*; a mid-based intrinsic may not be tradeable |
| 15 | Define Basis01 (bump object/units/sign) | PV change for +1bp increase in $b$, implemented as +1bp bump to $S_{\text{quoted}}$ holding constituents fixed; units $/\text{bp}$; positive for long protection |
| 16 | Why prefer proportional PSA adjustments? | Absolute bp shifts can distort tight names much more than wide names; proportional scaling better preserves relative risk ranking |
| 17 | Why is survival-probability PSA often faster than spread PSA? | It can avoid rebuilding full survival curves for every issuer on every iteration |
| 18 | What happens to intrinsic when dispersion increases? | It often falls relative to the simple average because wide names carry smaller RPV01 weights |
| 19 | Name two limits to basis arbitrage | Execution/market impact and margin/capital constraints (also contract mismatch and convergence uncertainty) |
| 20 | In stress, what can happen to basis? | Basis can widen when the index reprices faster than the constituent snapshot; convergence timing is uncertain |
| 21 | What P&L impact does a +3bp basis widening have on USD 100mm with RPV01=4.2? | Approx $100mm \times 3 \times 10^{-4} \times 4.2 \approx USD 126{,}000$ (long-protection index view) |

## Mini Problem Set

### Problems

1. Compute intrinsic spread for 4 names: spreads [40, 60, 80, 100] bp, RPV01 [4.5, 4.0, 3.5, 3.0] years.
2. Given quoted = 90 bp, intrinsic = 86 bp, compute basis and interpret.
3. Compute the upfront (dollars) for: $C=50$ bp, $S=80$ bp, $RPV01=4.2$ years, $N=USD 200\text{mm}$.
4. Why does a 1000 bp name get less weight in intrinsic than a 10 bp name?
5. Compute a bid/ask intrinsic band: 2 names, Name A bid/ask 90/110, Name B 10/12, both RPV01=4.0.
6. Explain how documentation differences (e.g., restructuring treatment) can create a systematic cross-index basis.
7. Design a proportional PSA for a 3-name index quoted at 60 bp, constituents [50, 60, 80] bp with equal RPV01.
8. What is the CS01 of a USD 50mm index position with RPV01 = 4.2?
9. Explain why PSA is not unique (different adjustment choices can match the index but produce slightly different adjusted curves).
10. Show that if all RPV01s are equal, intrinsic equals the simple average spread.
11. A basis trade is long index protection, short constituent protection, CS01-neutral. If basis tightens 2 bp, what is P&L on USD 100mm notional with RPV01=4.0?
12. How should intrinsic calculation change if the index has non-equal weights $w_m$?
13. Explain how a roll from an old to a new series can affect basis even if “credit unchanged.”
14. Describe a scenario where the index moves 20 bp before the constituent snapshot updates in a crisis.
15. Why must tranche models apply PSA before calibrating correlation?
16. Suppose basis is -2 bp before a sell-off and +8 bp during the sell-off. What driver could explain this move?
17. Compute a PSA adjustment ratio for a 4-name index: quoted spread 45 bp, constituent spreads [30, 40, 50, 60] bp, RPV01s [4.5, 4.2, 3.9, 3.6] years.

### Solution Sketches (Selected)

**1.** Intrinsic $=\frac{40(4.5)+60(4.0)+80(3.5)+100(3.0)}{4.5+4.0+3.5+3.0}=\frac{1000}{15}=66.67$ bp.

**3.** Upfront dollars $=N\cdot (S-C)\cdot 10^{-4}\cdot RPV01=200\text{mm}\cdot 30\cdot 10^{-4}\cdot 4.2=USD 2.52\text{mm}$.

**8.** $CS01\approx N\cdot RPV01\cdot 10^{-4}=50\text{mm}\cdot 4.2\cdot 10^{-4}=USD 21{,}000/\text{bp}$.

**4.** Intrinsic uses RPV01 weights. A very wide name has lower survival and a shorter expected premium-paying life, so its RPV01 is smaller and it receives less weight than its “count weight.”

**16.** A consistent explanation is a timing/liquidity mismatch: the index reprices quickly (macro hedge flows), while single-name quotes lag or become stale, so intrinsic (built from the snapshot) moves less than the quoted index.

## References

- Dominic O'Kane, *Modelling Single-name and Multi-name Credit Derivatives* (intrinsic index spread, index basis drivers, portfolio swap adjustment)
- John C. Hull, *Options, Futures, and Other Derivatives* (standardized CDS quoting; fixed coupon + upfront mechanics)
- John C. Hull, *Risk Management and Financial Institutions* (credit indices and risk measurement intuition)
- Salih N. Neftci, *Principles of Financial Engineering* (CDS standardization and points-upfront intuition)
- Damiano Brigo, Massimo Morini, and Andrea Pallavicini, *Counterparty Credit Risk, Collateral and Funding* (standard CDS conventions; fixed running fee + upfront context)
