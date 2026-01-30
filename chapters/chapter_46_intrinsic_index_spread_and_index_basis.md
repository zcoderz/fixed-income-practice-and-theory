# Chapter 46: Intrinsic Index Spread and Index Basis

---

## Introduction

The index trades at 85 bp, but the constituents imply 82 bp. Where's the 3 bp?

This question captures the essence of one of credit trading's most persistent puzzles: the **index basis**. When you price a CDS index by aggregating its single-name constituents, you get one number—the *intrinsic* spread. When you look at where the index actually trades in the market, you often get a different number—the *quoted* spread. The difference is the basis, and understanding it is essential for anyone managing credit portfolios or executing relative value trades.

The stakes are practical and immediate. Suppose you hedge a portfolio of single-name CDS positions with an index trade, believing you've neutralized your spread risk. If the index basis moves against you—if the index tightens while your single-names stay put—you've just taken an unexpected loss despite having "hedged." O'Kane addresses this directly: "when we look at the index spread quoted in the market, we often find that it is not the same as the intrinsic index spread." This basis arises from multiple sources, including documentation differences, liquidity premia, and technical supply-demand dynamics.

For those in middle office roles—risk, operations, or product control—who see P&L breaks when index hedges don't match constituent moves, this chapter explains the underlying mechanics. The intrinsic spread is what your model says the index is worth; the quoted spread is what the market says. When these diverge, someone's P&L breaks.

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

## Conventions & Notation

| Convention | Description |
|------------|-------------|
| **Instrument focus** | CDS portfolio indices (CDX / iTraxx) treated as equal-notional portfolios of single-name CDS unless explicitly stated |
| **Premium accrual** | Quarterly payments, Actual/360 |
| **Discounting** | $Z(t,u)$ is the risk-free discount factor from $t$ to $u$ |
| **Credit modeling** | Each name $m$ has default time $\tau_m$, recovery $R_m$, and survival probability $Q_m(t,u) = P(\tau_m > u \mid \mathcal{F}_t)$ |
| **Index basis sign** | $b(t,T) \equiv S_{\text{quoted}}(t,T) - S_{\text{intrinsic}}(t,T)$, so positive basis means index quoted wider than intrinsic |

| Symbol | Definition |
|--------|------------|
| $t$ | Valuation time |
| $T$ | Index maturity |
| $M$ | Number of index constituents |
| $C(T)$ | Contractual index coupon (bp/yr), fixed at series inception |
| $S_m(t,T)$ | Par CDS spread for name $m$ to maturity $T$ |
| $\text{RPV01}_m(t,T)$ | Risky PV01 (risky annuity) for name $m$ |
| $S_{\text{intrinsic}}(t,T)$ | Bottom-up intrinsic index spread |
| $S_{\text{quoted}}(t,T)$ | Market quoted index spread |
| $b(t,T)$ | Index basis = quoted − intrinsic |
| $U(t,T)$ | Upfront PV (per unit notional) |

---

## 46.1 The Intrinsic Spread: What the Index "Should" Trade At

### 46.1.1 The Economic Intuition

A CDS index is fundamentally a portfolio of single-name CDS contracts. If markets were frictionless—no documentation differences, no liquidity premia, no trading costs—the index spread should exactly equal some appropriately weighted average of its constituent spreads. This "fair value" is what we call the **intrinsic spread**.

O'Kane provides the foundational insight: "arbitrage pricing arguments allow us to value a CDS index in terms of the survival curves of the reference entities. We call this the intrinsic value since we have calculated the value of the index swap as the sum of the values of its constituent parts."

The intrinsic spread answers a specific question: if I wanted to replicate the index by trading each constituent CDS individually, what spread would I implicitly be paying (or receiving) on an aggregate basis?

### 46.1.2 From Constituent PVs to Intrinsic Value

To derive the intrinsic spread, we start from the present values of the protection and premium legs for each constituent. Under standard simplifying assumptions (equal notional weights $1/M$, fixed recovery rates, independence of default and interest rates), O'Kane derives the intrinsic valuation by pricing each leg separately.

For the **protection leg**, a credit event on reference entity $m$ results in an immediate loss of $(1-R_m)/M$ to the index buyer. O'Kane shows that the expected present value can be written in terms of the individual CDS spreads and risky PV01s:

$$\text{Index protection leg PV}(t) = \frac{1}{M} \sum_{m=1}^{M} S_m(t,T) \cdot \text{RPV01}_m(t,T)$$

For the **premium leg**, contractually, a credit event reduces the spread payments by factor $1/M$. The premium leg value is:

$$\text{Index premium leg PV}(t) = \frac{C(T)}{M} \sum_{m=1}^{M} \text{RPV01}_m(t,T)$$

The **intrinsic value** of a short protection position (premium leg minus protection leg) is therefore:

$$\boxed{V_{\text{intrinsic}}(t,T) = \frac{1}{M} \sum_{m=1}^{M} \left( C(T) - S_m(t,T) \right) \text{RPV01}_m(t,T)}$$

Equivalently, for a long protection position (O'Kane's Equation 10.6):

$$V_{I}(t) = \frac{1}{M} \sum_{m=1}^{M} \left( S_m(t,T) - C(T) \right) \cdot \text{RPV01}_m(t,T)$$

This formula has an elegant interpretation:
- $(S_m - C)$ is the "off-market" amount for name $m$—how much its par spread differs from the index coupon
- $\text{RPV01}_m$ is the present value of receiving 1 bp per year until default or maturity
- The product $(S_m - C) \times \text{RPV01}_m$ is the upfront PV of that name's contribution

**Sanity checks:**
- If $S_m = C$ for all names, intrinsic value is zero (the index is "at par")
- If spreads widen (holding coupon fixed), long protection value increases: $V_{\text{intrinsic}} \uparrow$

### 46.1.3 The Risky PV01 (RPV01)

The RPV01 is the key building block for CDS valuation. O'Kane defines it as "the time $t$ present value of a credit risky \$1 annuity which matures at time $T$." Under continuous premium payment approximation:

$$\boxed{\text{RPV01}_m(t,T) \equiv \int_t^T Z(t,u) \, Q_m(t,u) \, du}$$

This represents the present value of receiving $1 per year, continuously, until default or maturity—whichever comes first. The survival probability $Q_m(t,u)$ ensures we only count premium periods where the name survives.

**Unit analysis:**
- $Z(t,u)$ is unitless (discount factor)
- $Q_m(t,u)$ is unitless (probability)
- $du$ has units of years
- Therefore RPV01 has units of years—a "risky duration" of premium payments

For a name with higher default risk (lower survival probability), the RPV01 is lower because the expected premium-paying period is shorter. This has important implications for how we weight constituent spreads when computing intrinsic.

> **Desk Reality: How Traders Think About RPV01**
>
> When a trader says "this name has short duration," they often mean its RPV01 is low—either because it's near maturity or because it's distressed with high default probability. A 1000 bp name might have RPV01 of only 2 years versus 4.5 years for a 50 bp name at the same maturity. This "risky duration" is the natural unit for weighting credit exposure.

### 46.1.4 The Exact Intrinsic Spread Definition

The intrinsic spread $S_{\text{intrinsic}}$ is defined as the flat spread that, when applied to the index using its market-convention flat curve, produces the same PV as the bottom-up constituent calculation. O'Kane explains that "market convention is that the index curve $S_I(t,T)$ is flat. It is found by solving for the value of $S_I(t,T)$ at which a CDS contract with a coupon $C(T)$ has the same upfront value as the index."

Mathematically, we calculate the upfront value of the index using a flat index curve as:

$$U_I(t) = (S_I(t,T) - C(T)) \cdot \text{RPV01}_I(t,T)$$

Equating the intrinsic upfront and the index curve upfront (O'Kane's Equation 10.8):

$$\boxed{\frac{1}{M} \sum_{m=1}^{M}\left(S_{m}(t, T)-C(T)\right) \cdot \mathrm{RPV01}_{m}(t, T)=\left(S_{\mathrm{intrinsic}}(t, T)-C(T)\right) \cdot \mathrm{RPV01}_{I}(t, T)}$$

Here $\text{RPV01}_I$ is calculated using a flat index curve—a market convention for index pricing. Because $\text{RPV01}_I$ itself depends on the spread level, this equation is mildly nonlinear and requires solving iteratively (e.g., via Newton-Raphson or bisection).

### 46.1.5 The Operational Approximation: RPV01-Weighted Average

For practical purposes, O'Kane derives a close approximation that avoids iteration. If the individual spread curves are reasonably homogeneous, we can approximate:

$$\text{RPV01}_I(t,T) \simeq \frac{1}{M} \sum_{m=1}^{M} \text{RPV01}_m(t,T)$$

This allows cancellation of the coupon term, yielding O'Kane's Equation 10.9:

$$\boxed{S_{\text{intrinsic}}(t,T) \approx \frac{\sum_{m=1}^{M} S_m(t,T) \cdot \text{RPV01}_m(t,T)}{\sum_{m=1}^{M} \text{RPV01}_m(t,T)}}$$

**Derivation sketch:**
1. Start from the exact equation above
2. Approximate $\text{RPV01}_I \approx \frac{1}{M} \sum_m \text{RPV01}_m$
3. The coupon term cancels on both sides
4. Solve for $S_{\text{intrinsic}}$

**Why RPV01 weighting, not equal weighting?**

A simple average of constituent spreads would be wrong because it ignores the different "durations" of premium exposure across names. A name trading at 1000 bp has high default probability and short expected premium-paying life (low RPV01). Weighting by RPV01 reflects that we expect to pay spread to that name for less time.

O'Kane provides a mathematical intuition: "the RPVOI-weighted average spread... is a concave function and so its average is less than the simple average." This explains why intrinsic is typically below the simple average when spread dispersion exists.

### 46.1.6 The Concavity Effect: Jensen's Inequality Applied

> **Practitioner Note (extended from Claude knowledge):** O'Kane notes that the RPV01-weighted average is "concave" but doesn't provide the full mathematical derivation. Here we explain why this effect arises using Jensen's inequality.

The key insight is that RPV01 is a *decreasing, concave* function of spread. As spread increases, RPV01 falls (because default probability rises, shortening expected premium life). Moreover, this decrease is convex—the RPV01 of a 500 bp name is more than halfway between a 0 bp name and a 1000 bp name because default risk accelerates nonlinearly with spread.

Consider two names with spreads $S_1$ and $S_2$ where $S_2 > S_1$. Let $f(S) = \text{RPV01}(S)$ be the RPV01 as a function of spread. The intrinsic spread is:

$$S_{\text{intrinsic}} = \frac{S_1 \cdot f(S_1) + S_2 \cdot f(S_2)}{f(S_1) + f(S_2)}$$

The simple average is:

$$\bar{S} = \frac{S_1 + S_2}{2}$$

Because $f(S)$ is decreasing, the high-spread name gets *less* weight in the intrinsic calculation than equal weighting would give. The RPV01-weighted average is therefore pulled toward the low-spread name.

More formally, define weights $w_m = \text{RPV01}_m / \sum_k \text{RPV01}_k$. Since high-spread names have low RPV01, these weights underweight high-spread names relative to equal weights. The intrinsic spread is:

$$S_{\text{intrinsic}} = \sum_m w_m S_m < \frac{1}{M} \sum_m S_m = \bar{S}$$

when spread dispersion exists. This is a direct application of Jensen's inequality: the weighted average of a convex transformation (here, the identity function under weights that are inversely related to the argument) differs systematically from the equally-weighted average.

**Numerical demonstration (2-name toy example):**

| Name | Spread (bp) | RPV01 (years) | Weight |
|------|-------------|---------------|--------|
| A (Safe) | 50 | 4.5 | 4.5/5.0 = 90% |
| B (Distressed) | 1000 | 0.5 | 0.5/5.0 = 10% |

- **Simple average:** $(50 + 1000)/2 = 525$ bp
- **Intrinsic (RPV01-weighted):** $0.90 \times 50 + 0.10 \times 1000 = 45 + 100 = 145$ bp

The distressed name contributes 10% weight despite being 50% of the portfolio by count. The intrinsic spread (145 bp) is dramatically below the simple average (525 bp).

> **Deep Dive: The High-Yield Trap (Why Simple Average Lies)**
>
> Imagine a 2-name index:
> *   **Name A**: 50 bp (Safe). RPV01 ≈ 4.5 years.
> *   **Name B**: 5,000 bp (Distressed, near default). RPV01 ≈ 0.5 years.
>
> *   **Simple Average**: $(50 + 5000)/2 = 2,525$ bp.
> *   **Reality**: You will collect 50bp from Name A for 5 years, but 5000bp from Name B for only 6 months (then it dies).
> *   **Intrinsic**: The index is dominated by the *survivor* (Name A). The intrinsic spread will be much closer to Name A than the simple average suggests.
> *   **Lesson**: Never use simple average for high-dispersion portfolios.

O'Kane tested this approximation against the exact solution and found excellent accuracy for investment-grade portfolios. Table 46.1 summarizes his findings.

**Table 46.1: Approximation Accuracy (O'Kane Table 10.4)**

| Index | Coupon (bp) | Approx. $S_I$ (bp) | Exact $S_I$ (bp) | Difference (bp) |
|-------|-------------|-------------------|------------------|-----------------|
| iTraxx Europe Series 6 3Y | 20 | 13.928 | 13.928 | 0.000 |
| iTraxx Europe Series 6 5Y | 30 | 23.856 | 23.842 | 0.014 |
| iTraxx Europe Series 6 10Y | 50 | 41.920 | 41.855 | 0.065 |
| CDX NA IG Series 7 5Y | 40 | 35.556 | 35.540 | 0.016 |
| CDX NA IG Series 7 10Y | 65 | 61.403 | 61.363 | 0.040 |
| CDX NA HY Series 7 5Y | 325 | 226.166 | 224.666 | 1.500 |

*Source: O'Kane Ch 10, Table 10.4*

The high-yield case shows larger error (1.5 bp) due to higher spreads, higher spread dispersion, and large upfront value.

### 46.1.7 Intrinsic vs Simple Average: The Dispersion Effect

O'Kane demonstrates that spread dispersion systematically causes intrinsic to fall below the simple average. Table 46.2 shows this effect using real market data.

**Table 46.2: Intrinsic Spread vs Average Spread (O'Kane Table 10.5)**

| Index | Term | Intrinsic Spread (bp) | Average Spread (bp) | Std Dev (bp) |
|-------|------|----------------------|---------------------|--------------|
| CDX NA IG Series 7 | 3Y | 18.2 | 18.3 | 18 |
| CDX NA IG Series 7 | 5Y | 33.1 | 33.5 | 36 |
| CDX NA IG Series 7 | 7Y | 44.8 | 45.6 | 46 |
| CDX NA IG Series 7 | 10Y | 55.3 | 56.9 | 53 |
| iTraxx Europe Series 6 | 3Y | 16.2 | 16.2 | 12 |
| iTraxx Europe Series 6 | 5Y | 26.9 | 27.1 | 20 |
| iTraxx Europe Series 6 | 7Y | 36.2 | 36.5 | 25 |
| iTraxx Europe Series 6 | 10Y | 45.8 | 46.4 | 30 |
| CDX NA HY Series 7 | 3Y | 197 | 213 | 233 |
| CDX NA HY Series 7 | 5Y | 275 | 305 | 340 |
| CDX NA HY Series 7 | 7Y | 307 | 345 | 383 |
| CDX NA HY Series 7 | 10Y | 324 | 371 | 412 |

*Source: O'Kane Ch 10, Table 10.5*

O'Kane notes that "for the investment grade portfolios we see that the difference between the intrinsic spread and the average spread is small, of the order of 1-2 bp for the DJ CDX NA, rising to about 3 bp for the iTraxx Europe index." The difference is dramatically larger for high-yield: at 5Y, intrinsic is 275 bp versus average of 305 bp—a 30 bp gap. O'Kane explains this by noting "the very high standard deviation of the high-yield index spreads can be explained by a number of distressed credits with spreads greater than 1000 bp which were in this index."

> **Desk Reality: How Traders Actually Compute Intrinsic**
>
> In practice, traders rarely compute intrinsic manually. Vendor systems (Bloomberg CDSW, Markit, internal pricing engines) perform the RPV01 weighting automatically. However, understanding the mechanics matters when:
> - Debugging a P&L break between your model and the desk's
> - Evaluating a new name entering or exiting the index
> - Stress-testing a portfolio with distressed names
>
> Some systems use a flat 40% recovery for all names; others allow name-specific recoveries. The choice can matter when recoveries differ significantly (see Example 9).

---

## 46.2 The Index Basis: Definition and Measurement

### 46.2.1 Defining the Basis

The **index basis** is the difference between where the index actually trades and where it "should" trade based on constituents:

$$\boxed{b(t,T) = S_{\text{quoted}}(t,T) - S_{\text{intrinsic}}(t,T)}$$

With this sign convention:
- $b > 0$: Index quoted **wider** than intrinsic (index "cheap" vs constituents)
- $b < 0$: Index quoted **tighter** than intrinsic (index "rich" vs constituents)

O'Kane explicitly documents that "quoted index spreads often differ from intrinsic index spreads" and provides real market examples in Table 46.3.

> **Analogy: The ETF Mechanism**
>
> Think of the Index as an **ETF** and the Constituents as the **Stock Basket**.
>
> *   **Intrinsic Spread = NAV**: The Net Asset Value of the basket. This is the math.
> *   **Quoted Spread = ETF Price**: Where buyers/sellers actually trade the ticker. This is the market.
> *   **Basis = Premium/Discount**: Just like an ETF can trade at a premium to NAV during a buying frenzy, the CDS Index can trade wider than intrinsic during a credit panic.
> *   **Arbitrage**: If the gap gets too big, traders buy the Index and sell the Single Names (or vice versa) to close it, just like ETF Authorized Participants.

**Table 46.3: Index Basis Examples (O'Kane Table 10.6)**

| Index | Term | Quoted (bp) | Intrinsic (bp) | Basis (bp) |
|-------|------|-------------|----------------|------------|
| CDX NA IG Series 7 | 5Y | 34 | 33.1 | +0.9 |
| CDX NA IG Series 7 | 10Y | 55 | 55.3 | −0.3 |
| CDX NA HY Series 7 | 5Y | 276 | 275 | +1.0 |
| CDX NA HY Series 7 | 10Y | 315 | 324 | −9.0 |
| iTraxx Europe Series 6 | 5Y | 24 | 26.9 | −2.9 |
| iTraxx Europe Series 6 | 10Y | 42 | 45.8 | −3.8 |

*Source: O'Kane Ch 10, Table 10.6*

Notice that the basis can be positive or negative, varies across indices, and can differ by maturity within the same index.

### 46.2.2 Quoting Conventions Matter

The basis calculation requires consistency in how quoted and intrinsic spreads are measured. Investment-grade indices typically use a **spread quote** (market convention that prices the index "as if" a single-name CDS with flat curve). Some high-yield or crossover indices use a **bond price** convention where the upfront is computed as "price minus 100."

O'Kane notes that the bond price convention "avoids disagreement about the index PV01 needed to convert a spread quote into an upfront." When comparing basis across indices, ensure you're applying the same pricing convention to both sides.

### 46.2.3 Basis P&L Impact

> **Desk Reality: Why Basis Matters for P&L**
>
> On a $100mm index position with RPV01 of 4.2 years, a 3 bp adverse basis move costs:
>
> $$\Delta \text{PV} = 100,000,000 \times 3 \times 10^{-4} \times 4.2 = \$126,000$$
>
> This is pure basis P&L—distinct from parallel spread moves. A portfolio manager who hedges single-name exposure with an index can be perfectly CS01-neutral and still experience this P&L from basis volatility.
>
> **For middle office:** When you see a P&L break between index and constituent books that are supposedly hedged, basis movement is often the culprit. The basis is a distinct risk factor that survives CS01-neutral hedging.

---

## 46.3 Drivers of the Index Basis

The basis is not random noise—it reflects real economic frictions and structural differences between the index contract and its constituents. O'Kane identifies several systematic drivers.

### 46.3.1 Documentation Differences: Restructuring Clauses

This is perhaps the most quantifiable driver for North American indices.

O'Kane states directly: "In the case of the North American CDX index, the payment of protection on the index protection leg is only triggered when the credit event is a bankruptcy or failure to pay. Restructuring is not included as a credit event. These are called No-Re CDS. However, the market standard for CDS in the US is based on the use of the Mod-Re restructuring clause. Since No-Re spreads are typically about 5% lower than Mod-Re spreads, we have an immediate basis."

The logic is straightforward: restructuring is a "soft" credit event—following restructuring, debt can continue trading with a term structure of prices, unlike "hard" events (bankruptcy, failure to pay) where all debt typically trades at the same recovery. A contract that doesn't cover restructuring provides less protection, so it commands a lower spread.

O'Kane provides the spread ordering by clause in Chapter 5:

$$\boxed{S_{\text{Old-Re}} > S_{\text{Mod-Mod-Re}} > S_{\text{Mod-Re}} > S_{\text{No-Re}}}$$

**Illustrative impact:** If constituent spreads average 100 bp under Mod-Re and No-Re spreads are ~5% lower (O'Kane's rule-of-thumb), the No-Re equivalent is roughly 95 bp. This alone can create a few bp of intrinsic-vs-quoted discrepancy if one doesn't adjust.

> **Important caveat:** O'Kane notes a complexity: "While the European iTraxx indices include a restructuring credit event, the North American CDX index protection leg is only triggered by a bankruptcy or failure to pay." When computing intrinsic for CDX using standard Mod-Re single-name quotes, you must adjust for the documentation mismatch. The exact adjustment depends on the specific contract terms—always verify against current index documentation.

### 46.3.2 Liquidity and Supply-Demand Technicals

O'Kane identifies two liquidity-related basis drivers:

1. **Liquidity premium differential:** "The considerable size and liquidity of the CDS index market means that the CDS index spread embeds a lower liquidity risk premium than the less liquid CDS spreads."

2. **Index leads in stress:** "As the CDS index is more liquid, it tends to be the preferred instrument used by market participants to express a changing view about the credit market as a whole, or even one specific name in the index. As a result, the CDS index may be considered to lead the CDS market. This is especially true in a widening market where investors use long protection positions in the index to hedge illiquid long credit positions."

In stressed markets, hedging demand can concentrate in the liquid index and quoted index spreads can move faster than constituent quotes, creating basis moves. In benign markets, the index can trade tighter than a constituent-based intrinsic measure due to liquidity and technicals. The sign depends on how intrinsic is measured (mid vs executable), documentation alignment, and market microstructure.

**Bid-ask dispersion effect:** Even in calm markets, intrinsic computed from mid-market quotes may not be executable. If constituent bid-ask spreads are wide, the "executable intrinsic" (using bids or asks consistently) can differ from mid-intrinsic by several basis points.

### 46.3.3 Composition and Roll Effects

Indices roll semi-annually, refreshing the constituent list. O'Kane explains that rolling involves:
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

O'Kane notes that "a default on the CDS index should be offset by another index swap position of the same face value" for risk management purposes. These mechanics mean that indices with different default histories have different effective portfolios. Computing intrinsic requires knowing exactly which names remain and what the current effective notional is.

### 46.3.5 Limits to Basis Arbitrage

> **Practitioner Note (extended):** If the basis represents "free money," why doesn't arbitrage eliminate it? O'Kane discusses analogous frictions in the bond-CDS basis context (Section 5.6), and these apply with equal force to index basis arbitrage.

The ETF analogy suggests that if the index trades at a premium to intrinsic, you should buy the index and sell the constituents (or vice versa), capturing the basis. In practice, several frictions prevent this:

**1. Execution friction:** Trading 125 single-name CDS simultaneously is operationally difficult. Bid-ask spreads, market impact, and execution timing all erode the apparent arbitrage profit. O'Kane notes in the bond-CDS context: "it is difficult to assign degrees of importance to the individual factors as it is hard to tease them apart empirically."

**2. Capital constraints:** Basis trades require balance sheet. You must post margin on both the index and constituent legs, tying up capital. The return on capital may not justify the trade at small basis levels.

**3. Documentation mismatch creates real risk:** When CDX trades No-Re but constituents trade Mod-Re, you don't have a pure arbitrage—you have restructuring event risk. O'Kane explicitly warns: "the market standard for CDS in the US is based on the use of the Mod-Re restructuring clause." A Mod-Re restructuring event triggers your single-name short but not your index long, producing a real loss.

**4. Timing risk (convergence uncertainty):** Basis may not converge before the index rolls. If you put on a basis trade expecting convergence, you may need to roll the position—at a cost—or close at a loss if the basis persists.

**5. Funding and carry:** O'Kane notes for bond-CDS basis: "Default swaps are unfunded transactions and bonds are funded. For the same spread, CDS are favoured by investors who have funding costs above Libor while bonds are favoured by those who fund below." Similar funding dynamics affect index basis trades.

> **Desk Reality: Basis Arbitrage in Practice**
>
> Dedicated basis arbitrage desks exist, but they operate with tight risk limits and accept that:
> - Small basis moves can be hard to monetize after costs (bid-ask, execution, capital)
> - Basis can widen before converging
> - Basis can remain wide for extended periods in stress
>
> The basis is more like a "soft" arbitrage than a hard one—it rewards patient capital that can weather mark-to-market swings.

### 46.3.6 Basis Dynamics in Stress Periods

> **Practitioner Note:** O'Kane notes that the index can "lead" single-names in widening markets. A practical way to think about this is liquidity and measurement: indices are often the fastest macro hedge to trade, while single-name quotes can be stale or widen unevenly.

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

> **Practitioner Note (extended):** Traders hedging legacy positions sometimes face a choice: pay up for the matching (but illiquid) off-the-run series, or accept series basis risk with the liquid on-the-run. Most choose the latter, accepting that some basis P&L is the cost of liquidity.

**CDX vs iTraxx Geographic Basis:**

The two major index families have different compositions, documentation, and market dynamics:
- CDX (North American) uses No-Re documentation; iTraxx (European) includes restructuring
- Different constituent quality mixes (European vs North American credit profiles)
- Different investor bases and hedging patterns

Cross-index basis trades (long one, short the other) are occasionally implemented as macro bets on US vs European credit. However, these are not arbitrage—they are directional views on relative credit cycles with basis as the expression.

---

## 46.4 The Portfolio Swap Adjustment (PSA)

### 46.4.1 Why the Adjustment Is Needed

For many applications—particularly index option pricing and tranche valuation—practitioners need a model where intrinsic exactly equals quoted. O'Kane explains: "One way to ensure that the index swap spread equals the intrinsic swap spread is to see if there is an adjustment that can be made to the individual issuer curves which can enforce" the intrinsic-equals-quoted condition.

He further notes: "The exact nature of the adjustment is somewhat arbitrary. We choose to adjust the individual issuer curves rather than the index swap since the index swap is substantially more liquid and so their prices are more certain than the CDS quotes."

The adjustment modifies constituent curves (spreads or survival probabilities) so that when you compute intrinsic from these adjusted curves, you get exactly the market quoted spread. This ensures consistency between index and constituent pricing for products that depend on both (like tranches).

O'Kane states the calibration requirement clearly: "This is a prerequisite for those who wish to build models for products including options on CDS indices and especially the standard tranches which are based on these indices in a way which ensures that there is no built-in basis between the market value and intrinsic value."

### 46.4.2 Principles of the Adjustment

O'Kane emphasizes several desirable properties:

1. **Proportional, not absolute:** "We prefer a proportional adjustment of the spread rather than an absolute spread adjustment, i.e. we would rather increase all spreads by 1% than add say 5 bp to all spreads which would have a large effect on the credit risk of a name trading at 10 bp and a much smaller effect on another trading at 200 bp."

2. **Arbitrage-free:** The adjustment should not induce negative survival probabilities or other violations

3. **Preserve relative ranking:** The adjustment should not change which names are riskier than others

4. **Fast:** "Since it will become a preprocessing layer to much of the correlation product analytics which we will encounter later, it is important that the implementation is as fast as possible."

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

The algorithm solves for $\alpha(T)$ at each index maturity point using iteration:

1. Build all $M$ survival curves from original spreads
2. Calculate interpolated CDS spreads to index maturity dates
3. Initialize $\alpha = 1$
4. Apply $\alpha$ to spreads, rebuild curves, compute intrinsic
5. Update $\alpha$ based on how far intrinsic is from quoted
6. Iterate until convergence

O'Kane reports: "Most of the work of this iteration scheme is done in the first iteration since the dependence of the issuer PV01s on the spread is second order. In most cases we find that the algorithm converges with a tolerance in present value terms of the order of $O(10^{-8})$ within five iteration steps."

### 46.4.4 The Survival Probability Multiplier (Faster)

A more efficient approach adjusts survival probabilities directly, avoiding curve rebuilding at each iteration:

$$Q_{m}^{*}\left(t, T_{n}\right)=Q_{m}^{*}\left(t, T_{n-1}\right) \cdot Q\left(T_{n-1}, T_{n}\right)^{\alpha(n)}$$

This raises the forward survival probability to a power $\alpha(n)$. O'Kane explains the interpretation: "we can write this as $Q_m^*(t,T_n) = Q_m(t,T_{n-1}) \exp(-\alpha(n) \int_{T_{n-1}}^{T_n} h_m(s) ds)$. We see that $\alpha(n)$ is a simple multiplier on the forward default rate."

O'Kane reports this method is "about 50 times faster than the first method and typically takes less than a second for a 125-name portfolio using four maturity points."

### 46.4.5 Worked Example: PSA in Practice

O'Kane provides data from iTraxx Europe Series 6 (Table 10.7):

**Table 46.4: Portfolio Swap Adjustment Example**

| | 3Y Maturity | 5Y Maturity |
|-|-------------|-------------|
| **Index spread (bp)** | 12 | 24 |
| **Unadjusted intrinsic (bp)** | 16.2 | 26.9 |
| **Adjustment ratio** | 0.741 | 0.892 |

The adjustment ratios are less than 1 because the intrinsic exceeds the quoted—spreads must be scaled *down* to match. O'Kane notes: "We therefore need to scale down the hazard rate by some factor until the portfolio intrinsic equals that of the index."

Selected constituent adjustments from O'Kane Table 10.7:

| Credit | 3Y Unadj (bp) | 3Y Adj (bp) | Ratio | 5Y Unadj (bp) | 5Y Adj (bp) | Ratio |
|--------|---------------|-------------|-------|---------------|-------------|-------|
| Adecco S A | 23.2 | 17.1 | 0.737 | 38.8 | 34.4 | 0.89 |
| DaimlerChrysler AG | 34.4 | 25.4 | 0.737 | 52.9 | 44.6 | 0.84 |
| ITV PLC | 103.1 | 76.0 | 0.737 | 166.4 | 145.5 | 0.87 |

*Source: O'Kane Ch 10, Table 10.7*

Notice that the 3Y adjustment is uniform (0.737) while 5Y adjustments vary slightly by name due to different curve shapes. O'Kane explains: "It is not the same for all credits since the adjustment to the 3Y spreads has already been done and these proportional changes do not affect all issuer survival probabilities by the same amount."

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
> **Numerical example:** On a $50mm tranche with index RPV01 of 4.0 years and 5 bp basis:
> $$\text{Daily P\&L break risk} \approx 50{,}000{,}000 \times 5 \times 10^{-4} \times 4.0 = \$100{,}000$$
>
> This isn't a one-time error—it's a systematic mismatch that compounds over time.

---

## 46.5 Risk Measurement and Hedging Implications

### 46.5.1 Basis Exposure

A position in the index has exposure to basis moves independent of parallel spread moves. The first-order sensitivity is:

$$\frac{\partial \text{PV}}{\partial b} \approx \text{RPV01}_I(t,T) \times 10^{-4}$$

For a $100mm position with index RPV01 of 4.0 years, a 1 bp basis move produces approximately:
$$\Delta \text{PV} = 100{,}000{,}000 \times 4.0 \times 10^{-4} = \$40{,}000$$

### 46.5.2 P&L Decomposition: Index vs Constituents

Consider a book with both index and constituent positions. Total P&L decomposes as:

$$\Delta \text{PV} \approx \underbrace{N_I \cdot \text{CS01}_I \cdot \Delta S_{\text{quoted}}}_{\text{Index spread P&L}} - \underbrace{\sum_m N_m \cdot \text{CS01}_m \cdot \Delta S_m}_{\text{Constituent P&L}}$$

If you've hedged to be CS01-neutral ($N_I \cdot \text{CS01}_I = \sum_m N_m \cdot \text{CS01}_m$), parallel moves cancel. But residual P&L emerges from:
- **Basis moves:** $\Delta S_{\text{quoted}} \neq \sum_m w_m \Delta S_m$
- **Idiosyncratic moves:** Individual names move differently than the weighted average
- **Convexity:** Large moves where first-order hedging breaks down

### 46.5.3 Hedging an Index with Constituents

To hedge a long-protection index position with short-protection constituent trades, match CS01:

$$\text{Index CS01} = N_I \times 10^{-4} \times \text{RPV01}_I$$

$$\text{Constituent CS01} = \sum_m N_m \times 10^{-4} \times \text{RPV01}_m$$

For equal constituent notionals, set:
$$N_m = \frac{N_I \cdot \text{RPV01}_I}{\sum_m \text{RPV01}_m}$$

**Residual risks (failure modes):**
- Basis changes (quoted index moves without corresponding constituent move)
- Documentation mismatch (restructuring clause differences)
- Liquidity mismatch (constituents harder to trade at scale)
- Default mechanics (after default, notional and premium base change)

O'Kane addresses this directly: "if we wish to risk manage a CDS index with respect to its individual constituents, the index basis will present problems as we will need to account for it somewhere in our model."

### 46.5.4 Hedging Constituents with an Index (Proxy Hedge)

When single-name positions are illiquid, the index provides a liquid proxy hedge. This works best when:
- The book is "index-like" (similar composition)
- Basis is stable
- Idiosyncratic risk is accepted

In stressed markets, this hedge can break down exactly when you need it most—O'Kane notes the index may "lead" single-names, widening faster if hedging demand concentrates in the liquid instrument.

### 46.5.5 Systematic vs Idiosyncratic Delta

**Chapter 47** develops this framework in detail (following O'Kane's tranche delta treatment). The key insight is that spread risk decomposes into:

- **Systematic (index) delta:** Exposure to parallel moves in the index level
- **Idiosyncratic delta:** Exposure to individual name moves that don't affect the index average

For a pure index position, idiosyncratic delta is zero (by construction—you hold all names in weight). For a constituent position, both are nonzero. Basis risk emerges when systematic deltas are matched but basis itself moves.

> **Desk Reality: When to Accept Basis Risk**
>
> **Macro hedging:** Basis risk is often acceptable. You want systematic credit exposure hedged; if your IG portfolio correlates 0.9 with CDX IG, that's good enough for most purposes. Accept that basis P&L is the cost of using a liquid hedge.
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
| Name | $S_m \times \text{RPV01}_m$ |
|------|---------------------------|
| 1 | $50 \times 4.4 = 220$ |
| 2 | $60 \times 4.2 = 252$ |
| 3 | $70 \times 4.0 = 280$ |
| 4 | $80 \times 3.8 = 304$ |
| 5 | $90 \times 3.6 = 324$ |
| **Sum** | **1380** |

**Step 2: Compute denominator**
$$\sum_m \text{RPV01}_m = 4.4 + 4.2 + 4.0 + 3.8 + 3.6 = 20.0$$

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

1. **Documentation adjustment:** CDX is No-Re; constituent quotes are Mod-Re. O'Kane estimates No-Re ≈ 0.95 × Mod-Re.
   - Adjusted intrinsic: $75 \times 0.95 = 71.25$ bp
   - Documentation-adjusted basis: $70 - 71.25 = -1.25$ bp

2. **Liquidity premium:** Index more liquid → lower spread. Estimate 0.5 bp liquidity advantage.
   - Residual: $-1.25 + 0.5 = -0.75$ bp

3. **Measurement error / bid-ask:** Within typical 1 bp uncertainty

**Decomposition summary:**
| Driver | Contribution (bp) |
|--------|-------------------|
| Restructuring (No-Re vs Mod-Re) | -3.75 |
| Liquidity premium | +0.5 |
| Residual / noise | -0.75 |
| **Total basis** | **-5.0** |

**Conclusion:** Most of the apparent −5 bp basis is explained by the documentation mismatch (−3.75 bp from restructuring). After adjustment, the index trades only slightly tight to intrinsic.

---

### Example 4: Upfront Calculation (Index vs Coupon)

**Inputs:**
- Notional: $100mm
- Index coupon: $C = 60$ bp
- Quoted spread: $S_{\text{quoted}} = 72$ bp
- Index RPV01: 4.5 years

**Step 1: Spread difference**
$$S_{\text{quoted}} - C = 72 - 60 = 12 \text{ bp}$$

**Step 2: Upfront as percent of notional**
$$U = (12 \times 10^{-4}) \times 4.5 = 0.0054 = 0.54\%$$

**Step 3: Dollar upfront**
$$U_{\$} = 0.54\% \times \$100{,}000{,}000 = \$540{,}000$$

The protection buyer pays $540,000 upfront (because spreads are above coupon) plus running premium of 60 bp.

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

**Position:** Long protection on $100mm index with RPV01 = 4.1 years

**Index CS01:**
$$\text{CS01}_I = \$100mm \times 10^{-4} \times 4.1 = \$41{,}000/\text{bp}$$

**Hedge:** Short protection on 5 constituents with RPV01 = [4.4, 4.2, 4.0, 3.8, 3.6] years

For equal notional per name $N_m$:
$$5 \cdot N_m \times 10^{-4} \times \frac{20}{5} = 41{,}000$$
$$N_m \times 10^{-4} \times 4.0 = 8{,}200$$
$$N_m = \$20.5mm$$

**Hedge:** $20.5mm short protection per name (5 names, total $102.5mm notional) vs $100mm long protection index.

---

### Example 8: Basis Trade P&L Scenarios

**Position:** CS01-neutral (Example 7)
- Long protection index: CS01 = +$41k/bp
- Short protection constituents: CS01 = −$41k/bp

**Scenario (i): Parallel widening, no basis change**
- Index widens 20 bp, each name widens 20 bp
- Index P&L: $+20 \times 41k = +\$820k$
- Hedge P&L: $-20 \times 41k = -\$820k$
- **Net: $0** (systematic risk hedged)

**Scenario (ii): Pure basis move**
- Constituents unchanged, index tightens 10 bp
- Index P&L: $-10 \times 41k = -\$410k$
- Hedge P&L: $0$
- **Net: −$410k** (basis P&L)

**Scenario (iii): Idiosyncratic constituent widening**
- Index unchanged, Name 5 widens 100 bp (others flat)
- Index P&L: $0$
- Hedge P&L on Name 5: $-100 \times (20.5mm \times 10^{-4} \times 3.6) = -\$738k$
- **Net: −$738k** (idiosyncratic risk)

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

**Situation:** You hold $50mm long protection on CDX IG Series 37 (off-the-run). You want to hedge using the liquid on-the-run Series 38.

**Key differences between series:**
- Series 37 has 3 names that were replaced in Series 38 (fallen angels/upgrades)
- Series 38 has 6-month longer maturity (curves may slope)
- Liquidity: On-the-run bid-ask is typically tighter; off-the-run can be materially wider (illustrative)

**Hedge design:**
- Match CS01: Both series have RPV01 ≈ 4.1 years
- Hedge notional: $50mm Series 38 short protection

**Residual risks:**

1. **Composition mismatch:** The 3 replaced names create idiosyncratic exposure. If one of the dropped names widens 50 bp (it was dropped because it's deteriorating), your Series 37 position gains, but your Series 38 hedge doesn't respond.
   - Estimated impact: ~$50mm × (1/125) × 50 bp × 4.1 × 10⁻⁴ = ~$8,200

2. **Series basis:** If Series 37 trades at a different basis to its intrinsic than Series 38, you have basis mismatch.
   - If Series 37 basis is -2 bp and Series 38 is 0 bp, you're effectively short 2 bp of basis.

3. **Maturity mismatch:** Series 38 has 6 months more duration. If curves steepen (long end widens more), the hedge underperforms.

**P&L scenario:** Series 37 and 38 both widen 10 bp, but Series 37 basis widens 3 bp while Series 38 stays flat.
- Series 37 P&L: +10 bp × $41k/bp = +$410k (spread) + 3 bp × $41k/bp = +$123k (basis) = +$533k
- Series 38 P&L: -10 bp × $41k/bp = -$410k
- **Net: +$123k** (series basis slippage benefited you this time)

---

## 46.7 Practical Notes

### 46.7.1 Production Checklist

Before computing intrinsic and basis:

- **Index identity:** Series number, maturity, on-the-run vs off-the-run status
- **Constituent list:** Confirm which names are included (post any defaults)
- **Constituent spreads:** Consistent tenor, same valuation date, appropriate restructuring clause
- **Recovery assumptions:** Typically 40% for senior unsecured, 20% for subordinated
- **Discount curve:** Consistent across index and constituent pricing
- **Quoting convention:** Spread vs bond price; understand upfront conversion

### 46.7.2 Data Sourcing Considerations

> **Practitioner Note (extended):** Reliable intrinsic calculation requires accurate, timely constituent spreads.

**Where to get constituent spreads:**
- **Markit:** Industry-standard composite quotes, updated daily
- **Bloomberg CDSW:** Real-time indicative quotes from contributing dealers
- **DTCC/ICE:** Trade-level data (post-trade, with lag)
- **Internal pricing engines:** Calibrated to live dealer quotes

**Staleness flags:**
- If a single-name quote hasn't moved in 3+ days, it may be stale
- Illiquid names (off-index, subordinated) may have indicative-only quotes
- Verify against recent trade data if available

**Timing mismatch:**
- Index official close (typically 4:30 PM NY) may differ from constituent snapshot time
- Intraday intrinsic can move as constituent quotes update
- For EOD P&L, use consistent timestamps across index and constituents

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

## 46.8 Summary

1. A CDS index can be priced **bottom-up** by valuing constituent CDS and aggregating their PVs—this produces the **intrinsic value** and **intrinsic spread**

2. The intrinsic spread is approximated as an **RPV01-weighted average** of constituent spreads—not a simple average, because high-spread names have shorter expected premium-paying lives and lower RPV01s

3. The **concavity effect** (RPV01 weighting < simple average) arises from Jensen's inequality: RPV01 is a decreasing, concave function of spread, so high-spread names get underweighted

4. The market **quoted spread** often differs from intrinsic—this difference is the **index basis**

5. **Basis drivers** include:
   - Documentation differences (restructuring clauses—No-Re vs Mod-Re can explain ~5% spread difference)
   - Liquidity premia (index typically more liquid, may trade tighter)
   - Technical flows (hedging demand can push index wider in stressed markets)
   - Composition and roll effects
   - Cross-series and geographic factors

6. **Limits to basis arbitrage** prevent the basis from closing: execution friction, capital constraints, documentation mismatch risk, and timing uncertainty

7. **Basis behaves differently in stress**: Index typically leads constituents in widening markets; basis can blow out and remain wide for extended periods

8. The **portfolio swap adjustment** modifies constituent curves so that intrinsic exactly equals quoted—essential for index option and tranche pricing where model consistency is required

9. **Basis risk** is a real exposure: hedging an index with constituents (or vice versa) leaves residual P&L from basis moves, even when CS01 is matched

10. For model calibration in **tranche pricing** (Chapters 48-50), practitioners must first apply a PSA to ensure the constituent curves are consistent with quoted index levels

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **Intrinsic spread** | RPV01-weighted average of constituent spreads | The "fair value" of the index from bottom-up pricing |
| **Quoted spread** | Market-traded index level | What you actually pay/receive |
| **Index basis** | Quoted − Intrinsic | Measures deviation from replication value |
| **RPV01** | Risky PV01; PV of 1 bp premium until default/maturity | The correct weight for averaging spreads |
| **Concavity effect** | Intrinsic < simple average when dispersion exists | High-spread names have low RPV01, get less weight |
| **Portfolio swap adjustment** | Scaling constituent curves to force intrinsic = quoted | Required for tranche/option model calibration |
| **Restructuring clause** | Credit event definition (No-Re, Mod-Re, etc.) | Can create ~5% systematic spread difference |
| **Limits to arbitrage** | Frictions preventing basis closure | Explains why basis persists despite "free money" appearance |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is the intrinsic index spread? | The spread implied by pricing the index bottom-up from constituent CDS curves |
| 2 | What is the formula for the intrinsic spread approximation? | $S_{\text{intrinsic}} \approx \sum_m S_m \cdot \text{RPV01}_m / \sum_m \text{RPV01}_m$ |
| 3 | Why isn't intrinsic the simple average of constituent spreads? | High-spread names have lower RPV01 (shorter expected life) and should get less weight |
| 4 | Define index basis with sign convention | $b = S_{\text{quoted}} - S_{\text{intrinsic}}$; positive means index wider than intrinsic |
| 5 | What does positive basis indicate? | Index trades wider than constituent-implied value (index "cheap" vs constituents) |
| 6 | Name three drivers of index basis | (1) Documentation differences (restructuring), (2) Liquidity premium, (3) Technical flows |
| 7 | How much spread difference does restructuring clause typically create? | No-Re spreads ≈ 5% lower than Mod-Re spreads |
| 8 | What is RPV01 conceptually? | Risky annuity—PV of paying 1 bp/year until default or maturity |
| 9 | Why does the index sometimes "lead" single-names? | In stress, investors use liquid index to hedge, pushing it wider faster |
| 10 | What is the portfolio swap adjustment? | Scaling constituent curves so intrinsic exactly equals quoted index spread |
| 11 | Why is PSA needed for tranche pricing? | Model calibration requires no built-in basis between index and constituent values |
| 12 | What happens to index notional after a default? | Reduces by 1/M; future premiums shrink accordingly |
| 13 | What's the formula for index upfront? | $U = (S - C) \times \text{RPV01}_I$ |
| 14 | If all constituent spreads equal, what is intrinsic? | The common spread (weighted average reduces to simple average) |
| 15 | Why does off-the-run vs on-the-run hedging create risk? | Different constituent sets, liquidity levels, and possibly different basis across series |
| 16 | What is the typical basis range for IG indices? | ±1-5 bp in normal markets (based on O'Kane's examples) |
| 17 | What is the exact condition for intrinsic spread? | Intrinsic PV from constituents equals index PV under flat-curve convention |
| 18 | How does bid-ask dispersion affect intrinsic? | Creates a band of executable intrinsic values; mid may not be tradeable |
| 19 | What adjustment ratio is typical for PSA? | Often 0.7-1.0, depending on intrinsic vs quoted discrepancy |
| 20 | Why is survival probability adjustment faster than spread adjustment? | Avoids rebuilding all M survival curves in each iteration |
| 21 | What is the concavity effect in intrinsic calculation? | Intrinsic < simple average because RPV01 weighting underweights high-spread names |
| 22 | Name two limits to basis arbitrage | (1) Execution friction (125 names hard to trade), (2) Documentation mismatch creates real risk |
| 23 | How does basis typically behave during a credit crisis? | Widens as hedgers pile into liquid index; index leads constituents |
| 24 | What P&L impact does a 3 bp basis move have on $100mm position (RPV01=4.2)? | $100mm × 3 × 10⁻⁴ × 4.2 = $126,000 |

---

## Mini Problem Set

**Problems 1-9 include solution sketches; 10-17 are for practice.**

**1.** Compute intrinsic spread for 4 names: spreads [40, 60, 80, 100] bp, RPV01 [4.5, 4.0, 3.5, 3.0] years.

*Sketch:* Numerator = 40(4.5) + 60(4.0) + 80(3.5) + 100(3.0) = 180 + 240 + 280 + 300 = 1000. Denominator = 15. Intrinsic = 66.67 bp.

**2.** Given quoted = 90 bp, intrinsic = 86 bp, compute basis and interpret.

*Sketch:* $b = 90 - 86 = +4$ bp. Index wider than intrinsic (positive basis).

**3.** Compute upfront for: C = 50 bp, S = 80 bp, RPV01 = 4.2, N = $200mm.

*Sketch:* $U = (80-50) \times 10^{-4} \times 4.2 \times 200mm = 30 \times 0.0001 \times 4.2 \times 200mm = \$2.52mm$.

**4.** Why does a 1000 bp name get less weight in intrinsic than a 10 bp name?

*Sketch:* 1000 bp implies high default probability → short expected premium life → low RPV01 → less weight.

**5.** Compute bid/ask intrinsic band: 2 names, Name A bid/ask 90/110, Name B 10/12, both RPV01=4.0.

*Sketch:* Intrinsic(bid) = (90+10)/2 = 50 bp. Intrinsic(ask) = (110+12)/2 = 61 bp. Band = [50, 61] bp.

**6.** Explain why CDX basis typically differs from iTraxx basis by several bp.

*Sketch:* CDX uses No-Re, iTraxx includes restructuring. No-Re ≈ 5% lower spread, creating systematic difference.

**7.** Design a PSA for 3-name index quoted at 60 bp, constituents at [50, 60, 80] bp with equal RPV01.

*Sketch:* Unadjusted intrinsic = (50+60+80)/3 = 63.33 bp. Need α = 60/63.33 = 0.947. Adjusted: [47.4, 56.8, 75.8] bp.

**8.** What is the CS01 of a $50mm index position with RPV01 = 4.2?

*Sketch:* $50mm × 10^{-4} × 4.2 = \$21,000/bp$.

**9.** Explain why PSA is "somewhat arbitrary" per O'Kane.

*Sketch:* Multiple adjustment methods (spread multiplier, survival multiplier) give similar but not identical results. The choice affects which names are adjusted more. O'Kane states: "The exact nature of the adjustment is somewhat arbitrary."

**10.** Show that if all RPV01s are equal, intrinsic equals simple average spread.

**11.** A basis trade is long index protection, short constituent protection, CS01-neutral. If basis tightens 2 bp, what is P&L on $100mm notional with RPV01=4.0?

**12.** How should intrinsic calculation change if the index has non-equal weights $w_m$?

**13.** Explain how roll from old to new series can affect basis even if "credit unchanged."

**14.** Describe a scenario where the index leads constituents by 20 bp in a crisis.

**15.** Why must tranche models apply PSA before calibrating correlation?

**16.** Suppose basis is -2 bp before a sell-off and +8 bp during the sell-off. What driver could explain this move?

*Sketch:* The 10 bp widening of basis (from -2 to +8) is consistent with the index widening faster than the constituent-based intrinsic measure. A plausible driver is liquidity/technical hedging demand concentrating in the index while single-name quotes lag or become stale.

**17.** Compute the PSA adjustment ratio for a 4-name index: quoted spread 45 bp, constituent spreads [30, 40, 50, 60] bp, RPV01s [4.5, 4.2, 3.9, 3.6] years.

*Sketch:*
- Numerator: 30(4.5) + 40(4.2) + 50(3.9) + 60(3.6) = 135 + 168 + 195 + 216 = 714
- Denominator: 4.5 + 4.2 + 3.9 + 3.6 = 16.2
- Intrinsic: 714/16.2 = 44.07 bp
- Adjustment ratio: 45/44.07 = 1.021

Spreads must be scaled UP by 2.1% to match the quoted spread. Adjusted spreads: [30.6, 40.8, 51.1, 61.3] bp.

---

## References

- Dominic O'Kane, *Modelling Single-name and Multi-name Credit Derivatives* (intrinsic spread, basis drivers, portfolio swap adjustment / calibration)
- John C. Hull, *Risk Management and Financial Institutions* (credit indices and standardized CDS mechanics)

## Cross-References

- **Chapter 43** (CDS Risks): CS01 definition, VOD risk, spread duration
- **Chapter 45** (CDS Index Structure): Index naming, quoting conventions, lifecycle, roll mechanics
- **Chapter 47** (Index Hedging and RV): Applying basis concepts to hedge design, systematic vs idiosyncratic delta framework
- **Chapter 48-50** (Tranches): PSA required before tranche pricing—intrinsic must equal quoted for model consistency
