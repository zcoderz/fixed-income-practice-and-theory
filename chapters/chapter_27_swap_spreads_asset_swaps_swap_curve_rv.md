# Chapter 27: Swap Spreads, Asset Swaps, and Swap-Curve Relative Value

---

## Introduction

The 10-year swap spread is negative 20 basis points. The U.S. government—supposedly the risk-free benchmark—yields *more* than AA-rated bank credit. How is this possible?

This credit paradox has persisted since 2015 and fundamentally challenges the textbook interpretation of swap spreads as "credit spreads." If spreads measured pure credit, they could never be negative—no rational investor would accept lower compensation for taking on bank credit risk rather than holding Treasuries. Yet negative spreads exist, persist, and have become a structural feature of the post-crisis market.

Understanding this puzzle requires unpacking what swap spreads actually measure. At first glance, the answer seems obvious: swap rates are above (or below) Treasury yields of the same maturity, reflecting credit and liquidity differences. But this simple interpretation masks a web of complications. The Treasury yield you subtract may be distorted by on-the-run liquidity premiums and repo specialness. The swap rate reflects rolling short-term bank credit—not a static credit spread. And the spread itself conflates credit, supply/demand dynamics, bank capital constraints, and pension hedging demand in ways that can mislead the unwary trader.

This chapter unpacks the mechanics and interpretation of swap spreads and their close relative, the asset swap spread. Where swap spreads compare swap rates to government yields directly, asset swap spreads anchor individual bonds to the swap curve—providing a cleaner measure of a bond's value relative to bank-credit-based funding. Understanding the distinction matters because the two measures answer different questions and can move in opposite directions.

The chapter proceeds as follows. Section 27.1 defines swap spreads precisely and examines why benchmark choice matters. Section 27.2 explains the negative swap spread regime—the three structural drivers that pushed spreads negative post-2015. Section 27.3 introduces asset swaps as both a spread measure and a trading structure. Section 27.4 addresses the "premium bond trap"—a critical gotcha in asset swap analysis. Section 27.5 contrasts the zero-volatility spread (ZVS) with asset swap spreads. Section 27.6 develops the CDS-bond basis framework that links cash and derivative credit markets. Section 27.7 covers risk measurement, P&L attribution, and funding effects. Section 27.8 presents the swap-curve relative value framework. Section 27.9 addresses multi-curve considerations.

The reader should come away understanding: (1) why swap spreads can be negative and what that signals, (2) how to compute and interpret asset swap spreads using both Tuckman's and O'Kane's approaches, (3) the premium bond trap and how to avoid it, (4) the CDS-bond basis and its trading implications, and (5) how desks attribute P&L and manage funding costs.

---

## Notation for This Chapter

Before diving in, we establish notation that will remain consistent throughout.

### Rates and Units

| Symbol | Definition |
|--------|------------|
| Rates | In decimals (e.g., 5% = 0.05) |
| 1 bp | $= 0.0001$ |
| $f$ | Coupon frequency (payments per year) |
| $c/f$ | Coupon per period |
| $\Delta(t_{n-1}, t_n)$ | Year fraction (accrual factor) for period $n$ |

### Discount Factors and Zero Rates

| Symbol | Definition |
|--------|------------|
| $Z(0,t)$ | Discount factor to time $t$ |
| $r(0,t_n)$ | Discretely compounded zero rate to $t_n$ |
| Relation | $Z(0,t_n) = (1 + r(0,t_n)/f)^{-n}$ for $t_n = n/f$ |

### Swap Spread Notation

| Symbol | Definition |
|--------|------------|
| $S_T$ | Par swap rate for maturity $T$ |
| $y_T^{gov}$ | Government bond yield benchmark for maturity $T$ |
| $\text{SS}_T$ | Swap spread at maturity $T$: $\text{SS}_T = S_T - y_T^{gov}$ |

### Bond Prices

| Symbol | Definition |
|--------|------------|
| $P_c$ | Clean (quoted) bond price |
| $P$ | Full (dirty) bond price |
| $\text{AI}$ | Accrued interest |
| Relation | $P = P_c + \text{AI}$ |

### Asset Swap Spreads

| Symbol | Definition |
|--------|------------|
| $A$ | Par asset swap spread (decimal per year) |
| $A^*$ | Market asset swap spread; $A^* = A/P$ |
| $\theta$ | ZVS (zero-volatility spread) over the Libor discount rate |

### Other Notation

| Symbol | Definition |
|--------|------------|
| $P_{\text{Libor}}(0,T)$ | PV of bond's cashflows using Libor/swap discount factors |
| $\text{PV01}(0,T)$ | Discounted accrual sum: $\sum_{n=1}^{N} Z(0,t_n) \Delta(t_{n-1}, t_n)$ |
| $\text{DV01}$ | Dollar value of 1 bp move: $\text{DV01} \equiv -\Delta P / (10{,}000 \cdot \Delta y)$ |

---

## 27.1 Swap Spreads: Definition, Drivers, and Benchmark Dependence

### 27.1.1 The Basic Definition

The swap spread at maturity $T$ is the difference between the par swap rate and the government bond yield at that maturity:

$$\boxed{\text{SS}_T = S_T - y_T^{gov}}$$

This definition appears straightforward, but Tuckman emphasizes a critical practical issue: "A $T$-year swap matures in exactly $T$ years, but often there is no government bond with exactly $T$ years to maturity." By market convention, the "$T$-year swap spread" is quoted versus the $T$-year on-the-run government bond. This convention creates the first layer of complexity in interpreting swap spreads.

The maturity mismatch is not merely a technical detail. The on-the-run 10-year Treasury might have 9.8 years remaining; the swap has exactly 10 years. This small difference affects comparisons, particularly when the yield curve is steep.

### 27.1.2 Why Benchmark Choice Matters

The choice of government benchmark has a material impact on the quoted spread. On-the-run Treasury securities trade at premium prices (lower yields) due to liquidity advantages and special repo financing. Tuckman warns that "liquidity premiums and special financing have a large impact on the yield of on-the-run government bonds," meaning that "swap spreads quoted off on-the-run benchmarks can be misleading in 'the small.'"

Consider a concrete example. Suppose the 5-year par swap rate is 3.20% and two potential benchmarks exist:

| Benchmark | Yield | Swap Spread |
|-----------|-------|-------------|
| On-the-run 5Y Treasury | 2.95% | 25 bp |
| Fitted off-the-run curve | 3.05% | 15 bp |

The same swap rate produces swap spreads differing by 10 bp purely from benchmark choice. This is not a small discrepancy for relative value trading—10 bp might exceed the entire edge a trader is seeking to capture.

To address this issue, some desks compute swap spreads relative to fitted yields from an off-the-run government curve or explicitly adjust on-the-run yields for financing and liquidity effects. As Tuckman notes, "Both of these solutions require a good deal of subjective judgement," which is why adjusted swap spreads tend to be used by individual trading desks rather than being widely quoted in the marketplace.

> **Practical note:** When analyzing swap spread time series, be aware that the benchmark may change when a new on-the-run issue appears. This can create artificial jumps in the series that have nothing to do with credit or supply/demand dynamics.

### 27.1.3 Why Swap Spreads Are Not Pure Credit Spreads

A natural interpretation of swap spreads is that they measure the credit risk of the banking sector relative to government credit. While there is truth in this view, it oversimplifies the dynamics substantially. Tuckman provides the nuanced explanation:

> "While three-month LIBOR depends on the credit of the banking sector and swap rates depend on the evolution of three-month LIBOR, it is not true that the 10-year swap rate should equal the yield on a 10-year bond issued by a financially solid bank."

The key insight is that swap rates reflect *rolling* short-term bank credit—specifically, the three-month credit of banks that remain on the LIBOR polling list (or in the SOFR universe, the overnight credit of the repo market). This is fundamentally different from static term credit. As Hull's Risk Management text explains, the swap rate represents what a bank can expect to earn from "a series of short-term loans to AA-rated borrowers at LIBOR. It is sometimes referred to as a continually refreshed rate."

Since banks are dropped from the LIBOR panel after credit deterioration, LIBOR never fully reflected the credit of banks with very serious problems. A 10-year bank bond yield, by contrast, reflects the possibility that a particular bank might experience severe credit problems over the entire horizon.

Furthermore, swap spreads are contaminated by Treasury-side effects that have nothing to do with bank credit:

**Treasury scarcity.** When Treasury supply is perceived as scarce relative to demand, Treasury yields fall, widening swap spreads even if bank credit conditions are unchanged.

**Flight-to-quality flows.** During market stress, Treasury yields can drop sharply as investors seek safety, mechanically widening swap spreads.

**On-the-run specialness.** Repo financing advantages for on-the-run securities depress their yields below "fair value."

Tuckman documents this historically: "The fall in swap spreads in the early 1990s reflected the recovery of the banking sector from its problems in the 1980s. The rise in swap spreads in the late 1990s, on the other hand, can be best explained by a perceived scarcity in the supply of U.S. Treasury securities relative to demand."

The practical implication is that a widening swap spread can come from swap rates rising (bank credit deterioration or increased swap demand) *or* Treasury yields falling (flight-to-quality, specialness, supply scarcity)—and attributing the move correctly requires analysis beyond the spread itself.

### 27.1.4 Swap Spread Curve and Term Structure

Swap spreads can differ across maturities, forming a "swap spread curve." The slope of this curve—whether spreads are wider at the short end or long end—provides information about relative pricing across the term structure.

Consider a snapshot at two maturities using on-the-run benchmarks:

| Maturity | Swap Rate | OTR Treasury Yield | Swap Spread |
|----------|-----------|-------------------|-------------|
| 5Y | 3.20% | 2.95% | 25 bp |
| 10Y | 3.80% | 3.60% | 20 bp |

The swap spread curve slope from 5Y to 10Y is $20 - 25 = -5$ bp, meaning the curve is downward sloping (tightening with maturity) in this snapshot. Interpreting this slope requires caution because the drivers need not be "credit only." The Treasury-side benchmark can move for liquidity or specialness reasons that differ by maturity. The 5Y on-the-run might be more special than the 10Y on-the-run, artificially widening the 5Y swap spread relative to the 10Y.

> **Sanity check:** If the swap spread curve has an unusual shape (e.g., strongly inverted), investigate the repo specialness of each benchmark Treasury before drawing credit conclusions.

---

## 27.2 The Negative Swap Spread Regime

### 27.2.1 The Post-2015 Credit Paradox

Since 2015, U.S. swap spreads at longer maturities (10Y, 30Y) have frequently traded negative—meaning the "risk-free" Treasury yield exceeds the swap rate that embeds AA bank credit. This appears to violate the most basic credit logic: why would investors accept *less* compensation for taking on bank credit risk than for holding Treasuries?

The answer lies in recognizing that swap spreads are not pure credit spreads. Three structural forces combine to push swap spreads negative, and understanding them is essential for any modern practitioner.

### 27.2.2 Driver 1: Treasury Supply Surge

The first driver is the dramatic increase in Treasury supply since the financial crisis. Tuckman notes that swap spreads in the late 1990s widened due to "perceived scarcity in the supply of U.S. Treasury securities relative to demand." The post-2015 period represents the opposite phenomenon.

U.S. government deficits have produced enormous Treasury issuance—trillions of dollars annually. This supply must be absorbed by the market, pushing Treasury prices down (yields up) relative to swaps. The effect is particularly pronounced at the long end (10Y, 30Y) where duration is highest and supply is concentrated.

> **Desk Reality: The Supply Calendar**
>
> Rates desks watch the Treasury auction calendar obsessively. Before a large 30-year auction, swap spreads often tighten (become more negative) in anticipation of supply pressure on Treasury yields. After the auction, if demand is weak, spreads can tighten further. This is pure supply/demand—no credit story needed.
>
> "We're short swap spreads into the auction" means the trader expects Treasury yields to rise (prices fall) relative to swaps.

### 27.2.3 Driver 2: Bank Capital Constraints (SLR)

The second driver is bank capital regulation, specifically the Supplementary Leverage Ratio (SLR) introduced under Basel III. This regulation requires banks to hold capital against all assets—including low-risk assets like Treasury holdings and matched-book swap positions.

The critical implication: banks that historically arbitraged swap spread dislocations can no longer do so profitably. The classic trade—buy Treasuries, receive fixed on swaps, finance in repo—requires significant balance sheet. Under SLR, the capital cost of this trade often exceeds the spread earned, especially when spreads are narrow.

This breakdown of arbitrage allows swap spreads to remain persistently negative. In the pre-SLR world, a -10 bp swap spread would attract arbitrage capital until the spread normalized. Today, the spread can remain at -20 bp or wider because the arbitrage economics don't work.

> **Desk Reality: Balance Sheet as a Scarce Resource**
>
> Before SLR, swap spread arbitrage was a bread-and-butter trade for bank fixed income desks. Today, the conversation is different:
>
> *Trader:* "Swap spreads are -25 bp. Classic arb opportunity."
> *Risk manager:* "What's the balance sheet usage?"
> *Trader:* "We'd need $500mm notional for the Treasury leg."
> *Risk manager:* "That's $5mm of SLR capital for maybe $125k annual P&L before funding. Doesn't clear the hurdle."
>
> The arbitrage exists, but the capital constraint prevents exploitation.

### 27.2.4 Driver 3: Pension and LDI Hedging Demand

The third driver is structural demand from pension funds and insurance companies for long-duration hedges. These institutions have long-dated liabilities (pension obligations, annuity payouts) that they must hedge against interest rate declines. Their preferred instrument is receiving fixed on long-dated swaps.

This persistent one-way demand pushes swap rates down. Pension funds and insurers are natural receivers of fixed—they want to lock in rates on their liabilities. There is no equally large natural population of fixed payers at the long end, creating a structural imbalance.

The demand intensifies when rates fall (pension funding ratios deteriorate, requiring more hedging) and around liability-driven investment (LDI) rebalancing dates (quarter-end, year-end).

> **Practitioner Note:** The LDI demand effect is not directly documented in Tuckman or Hull but is well-known to practitioners as a key driver of long-end swap spread dynamics. UK gilt yields in 2022 spiked partly due to LDI-related forced selling, illustrating the magnitude of these flows.

### 27.2.5 The Three Drivers Combined

The negative swap spread regime reflects all three forces operating simultaneously:

| Driver | Effect on Treasury Yields | Effect on Swap Rates | Net Effect on Swap Spread |
|--------|--------------------------|---------------------|--------------------------|
| Treasury supply surge | Yields ↑ | Unchanged | Spreads ↓ (tighter/more negative) |
| SLR capital constraints | Arbitrage disabled | Arbitrage disabled | Spreads can stay negative |
| Pension/LDI demand | — | Swap rates ↓ | Spreads ↓ (more negative) |

The result is a structural regime where negative swap spreads are not an anomaly to be arbitraged away but a persistent feature of the market landscape.

### 27.2.6 Trading Implications

Understanding the negative spread regime has practical implications:

**Don't fade negative spreads on "credit logic."** The spread is not a mispricing—it reflects structural forces. Betting that "spreads must turn positive because credit" is a losing strategy.

**Watch the driver mix.** Swap spread moves can come from any of the three channels. Attributing a move to the wrong driver leads to incorrect positioning.

**Recognize regime dependence.** Pre-2015 intuition about swap spreads doesn't apply. Historical relationships (swap spread vs. credit indices, vs. equity vol) have shifted.

---

## 27.3 Asset Swaps: Structure, Pricing, and Conventions

### 27.3.1 Motivation: Anchoring Bonds to the Swap Curve

For short-term securities, TED spreads (based on Eurodollar futures rates) historically provided a convenient measure of value relative to LIBOR. But as Tuckman explains, "For longer-term bonds, market participants rely on asset swap spreads." The asset swap spread of a bond measures its value relative to the swap curve rather than to Treasuries, avoiding the idiosyncratic effects that contaminate Treasury-based spreads.

Tuckman defines the asset swap spread as "the spread such that discounting the bond's cash flows by swap rates plus that spread gives the bond price." This definition frames the asset swap spread as a *spread measure*—a way to quantify where a bond trades relative to the swap curve.

O'Kane provides a complementary perspective, framing the asset swap as a *contract structure* that combines a bond with an interest rate swap. He notes that since its inception in the early 1980s, the asset swap "has become an extremely important product for credit investors" and is "considered by some to be the first credit derivative." Hull's Risk Management text adds that asset swaps provide "direct estimates of the excess of bond yields over LIBOR/swap rates."

> **Concept: The Synthetic Floating Rate Note**
>
> Think of an Asset Swap (ASW) as a "currency converter for credit."
>
> *   **The Problem**: You want to buy Apple's credit risk, but Apple only issues fixed-rate bonds. You are a floating-rate investor (like a bank). You speak "floating," Apple speaks "fixed."
> *   **The Solution**: You buy the bond, but you wrap it in an interest rate swap.
>     *   **The Swap**: Takes the fixed coupons you receive from Apple and converts them into floating-rate payments (LIBOR/SOFR).
>     *   **The Result**: You now own a "synthetic floating-rate note" issued by Apple.
>     *   **The Spread**: The extra spread you get above LIBOR/SOFR is the "asset swap spread." It is the price of Apple's credit, stripped of interest rate duration.

### 27.3.2 Asset Swap Mechanics: The Par Asset Swap

In a par asset swap, an investor combines a bond purchase with an interest rate swap structured so that the combined package costs par. O'Kane describes the mechanics in detail:

**On settlement.** The asset swap buyer receives a fixed-rate bond with full price $P$ and pays par (i.e., 100% of face value). If $P \neq 1$, there is an upfront payment of $(1 - P)$ from the buyer (if the bond is at a discount) or to the buyer (if at a premium).

**Interest rate swap.** The buyer enters a payer swap with fixed leg payments matching the bond coupon schedule. On the floating leg, the buyer receives LIBOR plus the asset swap spread $A$.

**Cash flow netting.** The bond coupons received offset the fixed swap payments, leaving the buyer with a net position of receiving LIBOR + $A$ minus the cost of financing the bond in repo.

O'Kane emphasizes a critical detail: "It is essential to realise that these swap fixed leg payments are non-contingent, meaning that they are scheduled to be paid even if the bond defaults." This asymmetry—bond coupons cease on default but swap payments continue—is a key risk feature of asset swaps that distinguishes them from simple floating rate notes.

The implications are significant. If the bond defaults immediately after the asset swap begins, the investor loses both the bond (now worth only recovery $R$) and continues to owe fixed payments on the swap. This creates exposure beyond the simple credit risk of the bond itself.

### 27.3.3 The Par Asset Swap Spread Formula

To derive the formula for the par asset swap spread, we follow O'Kane's framework. Define the Libor-discounted value of the bond's fixed cashflows:

$$P_{\text{Libor}}(0,T) = \frac{c}{f} \sum_{n=1}^{N} Z(0,t_n) + Z(0,T)$$

This is the present value of the bond's fixed cashflows (coupons plus principal) discounted on the Libor/swap curve. Also define the PV01:

$$\text{PV01}(0,T) = \sum_{n=1}^{N} Z(0,t_n) \, \Delta(t_{n-1}, t_n)$$

which represents the present value of receiving 1 unit of spread per year on the floating leg. Note that $Z$ is dimensionless and $\Delta$ is in years, so PV01 has units of "discounted years."

O'Kane shows that the swap value at initiation is:

$$V(0) = 1 + A(0) \cdot \text{PV01}(0,T) - P_{\text{Libor}}(0,T)$$

The par asset swap structure requires $V(0) + P = 1$, which after rearrangement yields:

$$\boxed{A(0) = \frac{P_{\text{Libor}}(0,T) - P}{\text{PV01}(0,T)}}$$

O'Kane provides an elegant interpretation: "An asset swap is equivalent to going long a defaultable bond and short a risk-free bond with the same coupon schedule. The asset swap spread is then the amortised payment of the price difference over the life of the asset swap."

**Sign interpretation.** The formula provides clear intuition:
- If $P < P_{\text{Libor}}$ (bond trades cheap relative to the swap curve), then $A > 0$. The buyer receives a positive spread over LIBOR as compensation for taking the credit risk.
- If $P > P_{\text{Libor}}$ (bond trades rich relative to the swap curve), then $A < 0$. The buyer actually pays a spread below LIBOR.

### 27.3.4 Worked Example: Computing the Par Asset Swap Spread

Consider a 5-year bond with the following characteristics, adapted from O'Kane:

| Parameter | Value |
|-----------|-------|
| Coupon | 7.25% annual, semiannual payments |
| Full price | $94.38 per $100 face |

Using the Libor curve to discount the bond's fixed cashflows, O'Kane computes $P_{\text{Libor}} = 1.0876$ per dollar of face value, and $\text{PV01} = 4.4396$ discounted years.

Applying the formula:

$$A(0) = \frac{1.0876 - 0.9438}{4.4396} = \frac{0.1438}{4.4396} = 0.0324 = 324 \text{ bp}$$

**Interpretation.** This bond is significantly cheap relative to the swap curve—the investor would receive LIBOR plus 324 bp in the asset swap structure. O'Kane notes: "The asset swap spread is therefore a measure of the credit quality of the fixed rate bond. If the bond issuer has the credit quality of the AA commercial banking sector, then it will probably have a price close to $P_{\text{Libor}}$ and the asset swap spread will be close to zero."

**Repricing check.** We can verify: $V(0) = 1 + 0.0324 \times 4.4396 - 1.0876 = 1 + 0.1439 - 1.0876 = 0.0563$. Then $V(0) + P = 0.0563 + 0.9438 = 1.0001 \approx 1$ (the small error is rounding). ✓

### 27.3.5 The Market Asset Swap: An Alternative Structure

The par asset swap creates counterparty exposure when the bond trades away from par. As O'Kane explains: "Since the asset swap buyer pays par in exchange for a bond worth $P$, they are exposed to a default by the asset swap counterparty if the bond is trading at a discount."

Consider a bond trading at 95. The asset swap buyer pays 100 but receives a bond worth only 95. If the asset swap seller defaults immediately, the buyer has lost 5 points.

The market asset swap addresses this by setting the notional equal to the full bond price $P$ rather than par. O'Kane shows that the market asset swap spread $A^*$ relates to the par asset swap spread by:

$$\boxed{A^*(0) = \frac{A(0)}{P}}$$

For discount bonds ($P < 1$), the market asset swap spread is higher than the par spread because the floating leg notional is smaller. For premium bonds ($P > 1$), it is lower.

The mechanics are:
1. On settlement, the bond price $P$ is paid (not par)
2. The floating leg notional equals $P$
3. At maturity, the floating leg pays $P$ while the fixed leg pays 1, creating a net payment of $(P-1)$

O'Kane notes: "The ingenious feature of the market asset swap is that it has reversed both the timing and direction of the counterparty exposure." In the par structure, counterparty risk exists at initiation; in the market structure, it exists at maturity when the $(P-1)$ payment is due.

### 27.3.6 Mark-to-Market of an Asset Swap

O'Kane shows that the mark-to-market value of a par asset swap at time $t$ is:

$$\text{MTM}(t) = (A(0) - A(t)) \cdot \text{PV01}(t,T)$$

where $A(t)$ is the current asset swap spread for the bond. This formula follows from the fact that unwinding an asset swap involves entering an offsetting position at the current spread. The fixed legs and Libor payments cancel, leaving only the difference in spreads times the remaining PV01.

**Worked example.** Using O'Kane's numbers: if the asset swap was entered at $A(0) = 323.9$ bp on $10 million notional, and one year later the spread has tightened to $A(t) = 284$ bp with remaining PV01 of 3.622, the mark-to-market is:

$$\text{MTM} = (323.9 - 284.0) \text{ bp} \times 3.622 \times \$10\text{mm} = 39.9 \times 0.0001 \times 3.622 \times 10{,}000{,}000 = \$144{,}518$$

The position has profited because the credit quality of the issuer improved (spread tightened), as O'Kane notes: "The credit quality of the issuer has improved, as shown by the increase in the bond price, and the position has made money."

### 27.3.7 Asset Swaps in Tuckman's Framework

Tuckman presents asset swaps through a concrete trading example. Consider an investor who believes the FNMA 6.25s of May 15, 2029 are cheap relative to swaps at an asset swap spread of 15 bp. The investor executes:

1. Buy $100 million face of the bond at par
2. Finance the position in repo
3. Enter a swap paying 6.25% fixed and receiving LIBOR + 15 bp

The result is that "the coupon payments from the bond cancel the fixed payments to the swap," leaving the investor receiving "LIBOR plus 15 basis points minus the repo rate" so long as the agency does not default.

Tuckman emphasizes: "the asset swap spread is a measure of the credit risk of the bond relative to the swap curve while LIBOR minus repo is a measure of the credit risk of the swap curve." An investor who enters the asset swap earns the full credit spread of the bond—but loses everything if the bond defaults, since the swap obligations remain.

---

## 27.4 The Premium Bond Trap

### 27.4.1 The Problem: Par ASW Overstates Credit Spread for Premium Bonds

The par asset swap spread has a subtle but important flaw: it can significantly overstate the credit spread for bonds trading at a premium. This "premium bond trap" catches practitioners who mechanically compare par ASW spreads across bonds without adjusting for price effects.

The issue arises from the structure of the par asset swap. When a bond trades at a premium ($P > 100$), the asset swap buyer:
1. Pays par (100) but receives a bond worth $P > 100$
2. Receives an upfront payment of $(P - 100)$ from the swap counterparty

This upfront payment is effectively amortized into the spread over the life of the swap. For a high-coupon bond trading at 110, the "spread" includes not just credit compensation but also the amortization of the 10-point premium.

### 27.4.2 Intuition: Why Premium Bonds Look Artificially Wide

Consider two bonds from the same issuer with identical credit risk:

| Bond | Coupon | Price | Maturity | True Credit Spread |
|------|--------|-------|----------|-------------------|
| Bond A | 2% | 95 | 5Y | 100 bp |
| Bond B | 6% | 108 | 5Y | 100 bp |

Both bonds have the same credit spread in a Z-spread or CDS sense. But their par ASW spreads will differ:

- **Bond A (discount):** Par ASW must include compensation for the 5-point discount pulling to par, *reducing* the quoted spread
- **Bond B (premium):** Par ASW includes the benefit of the 8-point premium pulling to par, *increasing* the quoted spread

The premium bond appears wider on par ASW even though its credit risk is identical.

### 27.4.3 Worked Example: The Trap in Action

**Setup:**
- Issuer XYZ has two bonds outstanding
- Both have 5-year maturity and identical credit risk
- Swap curve is flat at 4%

| Bond | Coupon | Clean Price | YTM | Par ASW |
|------|--------|-------------|-----|---------|
| XYZ 3% | 3% | 95.50 | 5.00% | 80 bp |
| XYZ 7% | 7% | 112.00 | 4.80% | 120 bp |

A naive analyst looking only at par ASW would conclude that the 7% bond is "cheaper" (wider spread). But this is the premium bond trap—the 7% bond's wider par ASW reflects its premium price amortization, not superior value.

**The fix:** Use market ASW or Z-spread to compare:

$$A^*_{\text{3\%}} = \frac{80 \text{ bp}}{0.955} = 83.8 \text{ bp}$$
$$A^*_{\text{7\%}} = \frac{120 \text{ bp}}{1.12} = 107.1 \text{ bp}$$

Wait—the market ASW still shows the 7% bond wider. This is because even market ASW doesn't fully correct the issue. For true comparison, use Z-spread (which both bonds might show at ~100 bp) or the CDS spread.

> **Desk Reality: The RV Analyst's Checklist**
>
> When comparing bonds on ASW:
> 1. **Check dollar prices:** Premium vs. discount matters
> 2. **Use consistent measure:** Z-spread or CDS for apples-to-apples comparison
> 3. **If using par ASW:** Adjust for price/par difference explicitly
> 4. **Red flag:** If a high-coupon bond looks "cheap" on par ASW, verify with Z-spread before trading
>
> The classic mistake: "The 8% bonds are 30 bp wide to the 3% bonds on ASW—buy the 8s!" No—you may be buying the premium, not the spread.

### 27.4.4 When Par ASW Is Appropriate

Par ASW remains useful when:
- Comparing bonds at similar prices (all near par)
- The absolute spread level is more important than cross-bond comparison
- You're structuring an actual asset swap trade (where par-for-par mechanics apply)
- The analysis is paired with Z-spread or CDS-implied spread for validation

---

## 27.5 Zero-Volatility Spread: Definition and Relation to ASW

### 27.5.1 ZVS Definition

The zero-volatility spread (ZVS), also called the Z-spread, takes a different approach from asset swap spreads. O'Kane defines the ZVS as "the fixed spread adjustment to the Libor discount rate which reprices the bond."

In discrete compounding form:

$$P = \frac{c}{f} \sum_{n=1}^{N} \frac{1}{(1 + (r(0,t_n) + \theta)/f)^n} + \frac{1}{(1 + (r(0,t_N) + \theta)/f)^N}$$

Or in continuous compounding form:

$$P = \frac{c}{f} \sum_{n=1}^{N} Z(0,t_n) e^{-\theta t_n} + Z(0,T) e^{-\theta T}$$

where $\theta$ is the ZVS to be solved for numerically.

O'Kane prefers the term "zero volatility spread" to "option-adjusted spread" (OAS) for bonds without embedded options, noting that "the concept of the OAS has its origins in the callable bond market" and using the term for plain fixed-coupon bonds "can be confusing." Some practitioners also hedge using the ZVS rather than the yield because, as O'Kane notes, "the ZVS is preferred because it takes into account the term structure of the interest rate curve."

### 27.5.2 Conceptual Difference: ZVS vs. ASW

Although ASW and ZVS both measure a bond's spread to the swap curve, they are conceptually different:

| Aspect | ZVS | ASW |
|--------|-----|-----|
| **Mechanics** | Discounting spread: add $\theta$ to all discount rates | Swap-structure spread: spread on floating leg in a package |
| **What it values** | PV of bond cashflows with shifted discount curve | Floating leg receipts in swap-plus-bond package |
| **Solved from** | $P = \sum \text{DF}_{\text{shifted}} \times \text{CF}$ | $V(0) + P = 1$ in par asset swap structure |
| **Numerical method** | Root-finding (iterate $\theta$) | Direct formula |
| **Price sensitivity** | None—defined as the spread that reprices | Par ASW varies with price; market ASW adjusts |

For simple fixed-rate bonds under single-curve assumptions with a flat curve, the two measures are close. Tuckman notes that "only in the very special case of a flat swap rate curve does the asset swap spread equal the difference between the bond yield and the swap rate."

**When the measures diverge.** The ASW and ZVS can differ significantly when:

1. **Curve shape is non-flat.** ZVS shifts all discount rates uniformly by the same spread. ASW is determined by a PV01-weighted average that depends on the structure of the swap.

2. **Bonds trade away from par.** Par ASW includes price/par amortization effects; ZVS does not.

3. **Conventions differ.** Clean vs. dirty price inputs, par/par vs. market/market structures can create discrepancies.

4. **Multi-curve discounting.** OIS discounting with separate projection curves affects ASW and ZVS differently because ASW involves an actual swap contract.

### 27.5.3 Worked Example: ASW vs. ZVS Comparison

Consider a 3-year bond with semiannual coupons:

| Parameter | Value |
|-----------|-------|
| Coupon | 5% annual |
| Full price | $P = 1.0500$ |
| Discount factors | $Z = \{0.985, 0.970, 0.954, 0.938, 0.922, 0.905\}$ |

**Step 1: ASW calculation.**

First compute the Libor-discounted bond value:
$$P_{\text{Libor}} = 0.025 \times (0.985 + 0.970 + 0.954 + 0.938 + 0.922 + 0.905) + 0.905$$
$$P_{\text{Libor}} = 0.025 \times 5.674 + 0.905 = 0.14185 + 0.905 = 1.04685$$

Then compute PV01:
$$\text{PV01} = 0.5 \times 5.674 = 2.837$$

Apply the ASW formula:
$$A = \frac{1.04685 - 1.0500}{2.837} = \frac{-0.00315}{2.837} = -11.1 \text{ bp}$$

**Step 2: ZVS calculation.**

We need to find $\theta$ such that discounting with shifted factors reprices the bond at $P = 1.0500$. Since $P > P_{\text{Libor}}$, we need $\theta < 0$ (a negative spread increases PV by reducing discount rates). Iterative solution yields $\theta \approx -10.6$ bp.

**Comparison:**

| Measure | Value |
|---------|-------|
| ASW | $-11.1$ bp |
| ZVS | $-10.6$ bp |

The small difference (0.5 bp) arises from the structural distinction: ZVS applies a uniform shift to all discount rates, while ASW is determined through the PV01-weighted average implicit in the swap structure. For practical purposes, the measures are often close enough to be treated as equivalent for plain bonds near par.

---

## 27.6 The CDS-Bond Basis

### 27.6.1 Definition: Linking Cash and Derivative Credit Markets

The CDS-bond basis measures the difference between a credit derivative spread (CDS) and a cash bond spread (typically ASW or Z-spread):

$$\boxed{\text{CDS-Bond Basis} = \text{CDS Spread} - \text{ASW Spread}}$$

Hull's Risk Management text defines it as "the excess of the CDS spread over the asset swap spread." This basis connects two markets that should theoretically price the same credit risk—the CDS market and the cash bond market.

In a frictionless world, the basis should be zero: the cost of insuring against default via CDS should equal the credit spread earned by holding the bond. In practice, the basis fluctuates, sometimes significantly, creating relative value opportunities and revealing market stress.

### 27.6.2 Historical Behavior: Pre-Crisis vs. Crisis

Hull documents the dramatic shift in basis behavior around the 2008 financial crisis:

**Pre-2007:** The basis was typically small and positive. CDS spreads slightly exceeded ASW spreads, perhaps because CDS offers cleaner credit exposure (no funding, no rate risk). Hull notes: "Prior to the credit crisis that started in 2007, we would expect the CDS spread for a company to be a little higher than the yield spread calculated from its bonds."

**During crisis (2008-2009):** The basis became extremely negative—CDS spreads fell *below* ASW spreads, sometimes by 100+ bp. Hull explains: "During the credit crisis, the basis was at times very negative. This is because bond prices were very low and, for the reasons just mentioned, the CDS-bond basis trade was not possible." The arbitrage that should have closed the gap—buy the cheap bond, buy CDS protection—couldn't be executed because:
- Funding for bond purchases was unavailable or prohibitively expensive
- Balance sheet constraints prevented new positions
- Repo markets were dysfunctional

**Post-crisis:** The basis normalized but remains more volatile than pre-crisis, with periodic episodes of significant negativity during stress events.

### 27.6.3 Factors Affecting the Basis

Hull identifies six main factors that drive the CDS-bond basis:

| Factor | Effect on Basis | Explanation |
|--------|-----------------|-------------|
| **Funding costs** | Negative basis when funding expensive | Bond positions require financing; CDS does not |
| **Counterparty risk** | Positive basis (CDS wider) | CDS buyer faces protection seller credit risk |
| **Cheapest-to-deliver option** | Positive basis | CDS protection buyer can deliver cheapest bond |
| **Accrued interest treatment** | Varies | Settlement differences between bond and CDS |
| **Bond liquidity** | Negative basis when bonds illiquid | Bonds may trade cheap due to illiquidity |
| **Restructuring definition** | Positive basis for bonds at premium | CDS may pay par on restructuring; bond worth more |

### 27.6.4 The Basis Trade

When the basis is significantly non-zero, traders can attempt to capture it through a "basis trade":

**Negative basis trade (CDS tight relative to bonds):**
- Buy the bond (cheap)
- Buy CDS protection (relatively cheap)
- Net: long credit via bond, hedged via CDS
- Profit if basis normalizes (CDS widens or bond tightens)

**Positive basis trade (CDS wide relative to bonds):**
- Sell the bond short (rich)
- Sell CDS protection (relatively rich)
- Net: short credit via bond, offset by protection sale
- Profit if basis normalizes (CDS tightens or bond widens)

> **Desk Reality: Why Negative Basis Trades Failed in 2008**
>
> The classic negative basis trade looks like free money: buy a bond at Z+200, buy CDS at 150 bp, "lock in" 50 bp. But in 2008, this trade blew up spectacularly:
>
> 1. **Funding dried up:** Banks couldn't finance bond positions at reasonable rates. The "carry" evaporated.
> 2. **Mark-to-market pain:** As the basis widened further (to -100 bp, -150 bp), the position lost money on paper, triggering margin calls.
> 3. **Forced unwinds:** Levered investors had to sell bonds into illiquid markets, pushing the basis even more negative.
> 4. **The "trade" became one-way:** Everyone wanted out; no one could execute the arbitrage.
>
> Lesson: Basis trades require funding stability. In a crisis, "arbitrage" requires surviving the mark-to-market path to convergence.

### 27.6.5 Basis Trade P&L Attribution

For a negative basis trade (long bond, long CDS protection):

| Component | P&L Driver | Typical Magnitude |
|-----------|------------|-------------------|
| Carry (positive) | Coupon + ASW - CDS premium - funding | Running P&L |
| Rate P&L | Curve moves × DV01 (if not hedged) | Can dominate |
| Spread P&L | Bond spread change × spread DV01 | Main credit exposure |
| CDS P&L | CDS spread change × CDS DV01 | Offsets bond spread P&L |
| Basis P&L | (Bond spread - CDS spread) change | The trade's thesis |
| Funding P&L | Changes in repo/financing rates | Often overlooked |

The key insight: a "hedged" basis trade still has exposure to *changes in the basis itself*, plus funding rate risk.

---

## 27.7 Risk Measurement, P&L Attribution, and Funding

### 27.7.1 DV01 and Hedge Ratios

Asset swap positions carry interest rate risk because the bond DV01 typically differs from the swap fixed-leg DV01. Tuckman explains that this DV01 mismatch creates P&L from rate moves even in a "hedged" position.

The general DV01 definition from Chapter 11:

$$\text{DV01} = -\frac{\Delta P}{10{,}000 \times \Delta y}$$

which gives the dollar price change per 1 bp move in yield (per 100 face value).

To construct a DV01-neutral hedge, the swap notional should be sized according to:

$$\boxed{\text{Swap notional} = \text{Bond face} \times \frac{\text{DV01}_{\text{bond}}}{\text{DV01}_{\text{swap}}}}$$

**Example.** Consider a 3-year bond with coupon 5%, yield 5.20%, giving DV01 of 0.0263 per 100 face. If the corresponding swap fixed leg (using a 4.80% swap rate) has DV01 of 0.0350 per 100 notional, then the hedge ratio is:

$$\text{Swap notional fraction} = \frac{0.0263}{0.0350} = 0.751$$

So $100 million in bonds would be DV01-matched with approximately $75.1 million swap notional.

> **Practical warning:** DV01 matching creates a zero first-order exposure to parallel rate shifts, but residual risks remain—curve twists, convexity differences, and of course spread changes.

### 27.7.2 Spread Sensitivity

The sensitivity of the swap value to changes in the asset swap spread follows from differentiation of O'Kane's formula:

$$\frac{\partial V}{\partial A} = \text{PV01}$$

This means a 1 bp change in $A$ changes the swap value by approximately PV01 per unit notional. For a $100 million position with PV01 of 2.763 discounted years, a 1 bp change in ASW represents approximately:

$$\Delta V = 0.0001 \times 2.763 \times 100{,}000{,}000 = \$27{,}630$$

### 27.7.3 Comprehensive P&L Decomposition

An asset swap position's P&L can be decomposed into five main components:

**1. Rates P&L.** From parallel or non-parallel rate moves, captured through DV01 and key-rate exposures. If the position is DV01-hedged, this component should be small for parallel shifts but can be significant for curve twists.

**2. Spread P&L.** From changes in the asset swap spread, proportional to PV01. This is typically the intended exposure in an asset swap trade.

**3. Carry P&L.** The accrual of coupon income net of swap fixed payments and financing costs. For a DV01-matched position:
$$\text{Daily Carry} = \frac{\text{Bond Coupon} - \text{Swap Fixed Rate} + \text{ASW Spread} - \text{Repo Rate}}{360} \times \text{Notional}$$

**4. Rolldown P&L.** From the passage of time with an unchanged curve, as the bond "rolls down" the curve and the swap amortizes.

**5. Funding/Repo P&L.** From changes in repo financing rates or availability.

> **Desk Reality: The Funding Leg Matters**
>
> Tuckman notes that in his FNMA example, the investor's periodic receipts are "LIBOR plus 15 basis points minus the repo rate." Traders often describe this as "earning the LIBOR-repo spread" on top of the asset swap spread.
>
> **Example breakdown (annualized):**
> - Bond coupon: 5.00%
> - Swap fixed paid: 4.50%
> - ASW spread received: +50 bp
> - LIBOR received: 4.00%
> - Repo paid: 3.80%
>
> **Net carry:** (5.00 - 4.50) + (4.00 + 0.50 - 3.80) = 0.50 + 0.70 = 1.20% = 120 bp
>
> If repo suddenly widens 30 bp (special goes away, or funding stress), carry drops to 90 bp—a 25% reduction. This is why repo/funding is a critical component of ASW P&L.

### 27.7.4 Worked Example: P&L Scenarios

Consider a position that is long $100 million of a bond asset-swapped at $A_0 = 150$ bp with:
- Bond DV01: $26,300 per bp
- Swap hedge DV01: $26,300 per bp (DV01-matched)
- ASW spread sensitivity: $27,000 per bp (from PV01)

**Scenario 1: Parallel rate shift +10 bp, no ASW move**
- Bond P&L: $-10 \times \$26,300 = -\$263,000$
- Swap hedge P&L: $+10 \times \$26,300 = +\$263,000$
- Net P&L: $\approx \$0$ (rates hedged)

**Scenario 2: ASW tightens 20 bp, rates unchanged**
- Spread P&L: $+20 \times \$27,000 = +\$540,000$
- The trade profits from ASW normalization

**Scenario 3: Bond-specific spread widens 30 bp**
- Spread P&L: $-30 \times \$27,000 = -\$810,000$
- The trade loses from idiosyncratic widening

**Scenario 4: Repo rate spikes 50 bp**
- Funding cost increase: $+50 \times \$100\text{mm} / 10{,}000 = +\$500{,}000$ annually
- Over a quarter, this is ~$125,000 of additional cost
- The carry advantage erodes significantly

As Tuckman emphasizes, even positions framed as "collect the spread" can experience significant mark-to-market volatility from spread changes, potentially forcing position reductions at the worst time.

### 27.7.5 Repo Effects in ASW Trades

Repo rates directly affect ASW carry and can vary for several reasons:

| Repo Effect | Impact on ASW P&L |
|-------------|-------------------|
| **General collateral (GC) rate moves** | Higher GC = lower carry |
| **Bond goes special** | Lower repo = higher carry (bond cheaper to finance) |
| **Fails and scarcity** | Can make repo unpredictable |
| **Quarter-end balance sheet constraints** | Repo rates spike; carry compresses |
| **Credit events affecting collateral eligibility** | May lose repo access entirely |

> **Practitioner Note:** Repo market dynamics are covered in Chapter 9. For ASW positions, the key insight is that "earning LIBOR + spread" is net of repo—and repo can move significantly, especially during stress.

---

## 27.8 Swap-Curve Relative Value Framework

### 27.8.1 The RV Trader's Perspective

Swap-curve relative value is not a single formula but a *practice*—the comparison of bond value to the swap curve rather than to Treasuries. Tuckman motivates this approach: market participants use asset swap spreads and similar measures for longer-term bonds "to measure value relative to curves not contaminated by individual security effects."

The key questions in swap-curve RV are:
- Is this bond cheap or rich relative to the swap curve?
- How does its ASW compare to similar bonds or its own history?
- What would need to happen for the ASW to normalize?

### 27.8.2 Rich/Cheap Analysis

A bond is considered "cheap on ASW" when its asset swap spread is wide relative to:
- Historical levels for that issuer
- Comparable bonds in the same sector
- The spread implied by CDS levels (the "basis")

Conversely, a bond is "rich on ASW" when its spread is tight relative to these benchmarks.

O'Kane provides the intuition: "The asset swap spread is therefore a measure of the credit quality of the fixed rate bond. If the bond issuer has the credit quality of the AA commercial banking sector, then it will probably have a price close to $P_{\text{Libor}}$ and the asset swap spread will be close to zero."

For investment-grade issuers, ASW spreads are typically positive but modest. For high-yield issuers, ASW spreads can be hundreds of basis points. Negative ASW spreads signal that the market views the issuer as *better* than the AA banking sector—rare but not impossible for sovereigns or supranationals.

### 27.8.3 Tuckman's Spread-of-Spreads Trade

Tuckman's case study of on-the-run versus off-the-run five-year Treasuries illustrates how desks use swaps for relative value. The trade setup:

- The OTR 5-year was 91.3 bp rich to swaps
- The old 5-year was 75.4 bp rich to swaps
- The "spread of spreads" was $91.3 - 75.4 = 15.9$ bp

The trader believed this spread of spreads was too wide (the OTR was "too rich relative to the old five-year"). The trade involved:

1. Sell $100 million OTR 5-year
2. Buy repo to the forward date
3. Receive fixed on a DV01-matched swap
4. Buy a DV01-matched position in the old 5-year
5. Sell repo on the old 5-year
6. Pay fixed on a DV01-matched swap

Tuckman explains the key insight: by DV01-matching each leg, "the combination of the two asset swap trades generates P&L only from movements in the spread of spreads and not from equal moves of each asset swap spread."

The trade expresses a pure view on the relative value of the two securities, isolating the technical premium for on-the-run status.

### 27.8.4 What Drives ASW Changes

Asset swap spreads can widen or tighten for several reasons:

| Driver | Effect on ASW |
|--------|---------------|
| Issuer credit deterioration | Widens |
| General flight-to-quality | Widens (for non-government issuers) |
| Bond-specific supply pressure | Widens |
| Issuer credit improvement | Tightens |
| Strong bid for the sector | Tightens |
| Swap curve steepening/flattening | Can affect ASW through PV01 changes |
| Pension hedging flows | Can affect long-end swap rates, impacting ASW |

---

## 27.9 Multi-Curve Considerations

### 27.9.1 The Shift to OIS Discounting

Modern practice separates the discount curve (typically OIS-based for collateralized trades) from the projection curve (LIBOR or SOFR forward rates). This multi-curve framework, discussed in Chapters 18-22, affects asset swap spread calculations because:

1. The discount factors $Z(0,t)$ come from the OIS curve
2. The forward rates for the floating leg come from the projection curve
3. The relationship between $P_{\text{Libor}}$ and the bond price changes

A full multi-curve ASW calculation requires specifying:
- The discount curve (OIS)
- The projection curve (LIBOR/SOFR)
- Collateral/CSA terms
- The instrument set used to build each curve

> **I'm not sure** we can rigorously recompute ASW under OIS discounting without specifying (i) the projection curve, (ii) collateral/CSA terms, and (iii) the exact market instrument set used to build OIS vs LIBOR curves. The books confirm multi-curve separation (Andersen & Piterbarg Vol 1, Hull), but the full desk convention set varies by institution and is not pinned down here.

### 27.9.2 Sensitivity to Discounting Assumptions

Changing from legacy single-curve Libor discounting to OIS discounting can materially change ASW levels. As a sensitivity experiment, consider the bond from Section 27.5.3 with slightly higher OIS discount factors (reflecting OIS below LIBOR):

| Curve | Sum of $Z$ | PV01 | $P_{\text{disc}}$ | ASW |
|-------|-----------|------|-------------------|-----|
| Libor | 5.674 | 2.837 | 1.04685 | $-11.1$ bp |
| OIS (toy) | 5.688 | 2.844 | 1.0512 | $+4.2$ bp |

The ASW flips from negative to positive purely from the discounting change. This illustrates why stating the discounting convention is essential for any precise ASW calculation.

**Practical implication.** When comparing ASW levels across different sources (Bloomberg, Reuters, internal systems), verify that the discounting methodology is consistent. A 5 bp difference might reflect methodology, not relative value.

---

## 27.10 Worked Examples: Consolidated

The following examples consolidate and extend the calculations developed throughout the chapter.

### Example A: Swap Spread Computation with Benchmark Sensitivity

**Given:**
- 5Y par swap rate: $S_5 = 3.20\%$
- On-the-run 5Y yield: $y_5^{\text{OTR}} = 2.95\%$
- Fitted off-the-run yield: $y_5^{\text{fit}} = 3.05\%$

**Compute:**
$$\text{SS}_5^{\text{OTR}} = 3.20\% - 2.95\% = 25 \text{ bp}$$
$$\text{SS}_5^{\text{fit}} = 3.20\% - 3.05\% = 15 \text{ bp}$$

The same swap rate produces swap spreads differing by 10 bp from benchmark choice, illustrating Tuckman's warning about on-the-run distortions.

### Example B: ASW Cashflow Setup

**Given (per $100 notional):**
- Bond coupon: 5% annual, semiannual payments
- ASW spread: $A = 20$ bp
- LIBOR resets: $L = \{3.00\%, 3.20\%, 3.10\%\}$ (semiannual)
- Repo rate: $r_{\text{repo}} = 2.70\%$

| Date | Bond coupon | Swap fixed paid | Swap float received | Repo paid | Net |
|------|-------------|-----------------|---------------------|-----------|-----|
| 0.5y | +2.50 | $-2.50$ | $(3.00 + 0.20) \times 0.5 = +1.60$ | $-1.35$ | +0.25 |
| 1.0y | +2.50 | $-2.50$ | $(3.20 + 0.20) \times 0.5 = +1.70$ | $-1.35$ | +0.35 |
| 1.5y | +2.50 | $-2.50$ | $(3.10 + 0.20) \times 0.5 = +1.65$ | $-1.35$ | +0.30 |

The net cashflow reflects "(LIBOR + ASW − repo)" as Tuckman describes.

### Example C: Full ASW Calculation with Repricing Check

**Given:**
- 3Y bond, semiannual 5% coupon
- Full price $P = 1.0500$
- Discount factors $Z = \{0.985, 0.970, 0.954, 0.938, 0.922, 0.905\}$

**Step 1:** $P_{\text{Libor}} = 0.025 \times 5.674 + 0.905 = 1.04685$

**Step 2:** $\text{PV01} = 0.5 \times 5.674 = 2.837$

**Step 3:** $A = (1.04685 - 1.0500)/2.837 = -11.1$ bp

**Repricing check:** $V(0) = 1 + (-0.00111) \times 2.837 - 1.04685 = 1 - 0.00315 - 1.04685 = -0.05$. Then $V(0) + P = -0.05 + 1.05 = 1.00$ ✓

### Example D: Market Asset Swap Conversion

Using Example C with $A = -11.1$ bp and $P = 1.0500$:

$$A^* = \frac{A}{P} = \frac{-0.00111}{1.0500} = -0.00106 = -10.6 \text{ bp}$$

The market asset swap spread is slightly lower in magnitude because the floating notional is larger (premium bond).

### Example E: DV01-Matched Hedge Construction

**Given:**
- Bond DV01: 0.0263 per 100 face
- Swap fixed leg DV01: 0.0350 per 100 notional

**Hedge ratio:**
$$\text{Swap notional} = \$100\text{mm} \times \frac{0.0263}{0.0350} = \$75.1\text{mm}$$

### Example F: Premium Bond Trap Illustration

**Given two bonds from same issuer:**

| Bond | Coupon | Price | 5Y Maturity | Z-Spread |
|------|--------|-------|-------------|----------|
| A | 2% | 95 | 5Y | 100 bp |
| B | 6% | 108 | 5Y | 100 bp |

**Par ASW calculation (simplified):**
- Bond A: Par ASW ≈ Z-spread adjusted for discount = ~85 bp
- Bond B: Par ASW ≈ Z-spread adjusted for premium = ~130 bp

**Trap:** Bond B looks 45 bp "cheaper" on par ASW, but has identical credit risk. Use Z-spread for apples-to-apples comparison.

---

## Practical Notes

### Convention Checklist

**Swap Spread:**
- [ ] Government benchmark specified (on-the-run vs. fitted/off-the-run)
- [ ] Swap rate confirmed as par swap rate
- [ ] Swap curve source identified

**Asset Swap Spread:**
- [ ] Par/par vs. market/market convention stated
- [ ] Bond price input specified (clean $P_c$ vs. full $P$)
- [ ] Settlement and upfront payment handling for off-par bonds
- [ ] Discounting curve specified (legacy Libor vs. OIS)

**CDS-Bond Basis:**
- [ ] Bond spread measure specified (ASW vs. Z-spread)
- [ ] CDS contract terms noted (restructuring clause, currency)
- [ ] Funding assumption stated

### Common Pitfalls

| Pitfall | Description |
|---------|-------------|
| Mixing swap spread with ASW | Swap spread is (swap − gov); ASW is (bond vs. swap curve)—not identical |
| ZVS vs. ASW confusion | Different constructions (discounting spread vs. swap-package spread) |
| Clean/dirty mismatch | Using clean price where full price is required (or vice versa) |
| Ignoring default terms | Bond default does not extinguish swap fixed payments |
| Premium bond trap | Par ASW overstates spread for premium bonds |
| Forgetting basis and funding | Treasury specialness affects swap spreads; repo levels affect realized carry |
| Compounding mismatches | Semiannual vs. money-market conventions across curves |
| Assuming negative spreads are "wrong" | Post-2015, negative swap spreads are structural, not anomalies |

### Verification Tests

**Repricing check.** For a par asset swap, verify $V(0) + P = 1$.

**Sign check.** From $A = (P_{\text{Libor}} - P)/\text{PV01}$, if bond price $P$ falls (bond cheapens), $A$ should rise (widen).

**Unit check.** DV01 is per-100 price points; convert to dollars by $\text{Notional} \times \text{DV01} / 100$.

**Cross-check spreads.** If par ASW differs dramatically from Z-spread, investigate price/par effects.

---

## Summary

This chapter has developed the framework for understanding swap spreads, asset swaps, and swap-curve relative value:

1. **Swap spreads** ($\text{SS}_T = S_T - y_T^{gov}$) compare swap rates to government yields but are contaminated by on-the-run liquidity and specialness effects.

2. **Swap spreads are not pure credit spreads**—they reflect rolling bank credit (a "continually refreshed rate"), Treasury supply/demand, bank capital constraints, and pension hedging demand jointly.

3. **Negative swap spreads** have been structural since 2015, driven by Treasury supply, SLR capital constraints, and pension/LDI demand. Don't fade them on "credit logic."

4. **Asset swap spreads** anchor bonds to the swap curve, measuring value relative to bank-credit-based funding. The formula $A = (P_{\text{Libor}} - P)/\text{PV01}$ provides the par ASW spread.

5. **The premium bond trap**: Par ASW overstates credit spread for premium bonds; use Z-spread or market ASW for comparisons.

6. **Market asset swaps** use notional equal to bond price, with $A^* = A/P$, reversing the timing and direction of counterparty exposure.

7. **ZVS** is a discounting spread that differs structurally from ASW, though both measure bond value relative to swaps.

8. **The CDS-bond basis** (CDS spread minus ASW spread) links cash and derivative credit markets; negative basis signals funding stress.

9. **P&L attribution** must include rates, spread, carry, rolldown, and funding components. Repo effects can be material.

10. **Default asymmetry** is crucial: bond defaults stop coupons but swap obligations continue, creating asymmetric exposure.

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| Swap spread | $S_T - y_T^{gov}$ | Broad swaps vs. gov indicator, but benchmark-sensitive |
| Negative swap spread regime | Swap rate < Treasury yield | Post-2015 structural feature; not an anomaly |
| Three drivers of negative spreads | Treasury supply, SLR, pension demand | Explain why negative spreads persist |
| Asset swap spread | Spread on floating leg making bond+swap = par | Bond valuation vs. swap curve |
| Premium bond trap | Par ASW overstates spread for premium bonds | Use Z-spread for cross-bond comparison |
| $P_{\text{Libor}}$ | PV of bond cashflows on swap curve | Reference for cheap/rich determination |
| PV01 | Discounted accrual sum | Denominates spread formulas and sensitivities |
| ZVS | Discounting spread over Libor curve | Alternative spread measure with different mechanics |
| Market ASW | $A^* = A/P$ | Reduces counterparty risk for off-par bonds |
| CDS-bond basis | CDS spread − ASW spread | Links cash and derivative credit markets |
| Rolling credit | Swap reflects continually refreshed short-term credit | Why swap ≠ term bank bond yield |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | Define swap spread at maturity $T$. | $\text{SS}_T = S_T - y_T^{gov}$, usually vs. on-the-run gov yield |
| 2 | Why is on-the-run benchmarking problematic? | On-the-run yields include liquidity premiums and special financing effects |
| 3 | What does "continually refreshed rate" mean for swaps? | Swap rates reflect rolling short-term bank credit, not static term credit |
| 4 | Name three drivers of negative swap spreads post-2015. | Treasury supply surge, SLR capital constraints, pension/LDI hedging demand |
| 5 | Why can't banks arbitrage negative swap spreads? | SLR capital requirements make the arbitrage uneconomic |
| 6 | Write the par ASW spread formula. | $A = (P_{\text{Libor}} - P)/\text{PV01}$ |
| 7 | What is the premium bond trap? | Par ASW overstates credit spread for bonds trading above par |
| 8 | How to avoid the premium bond trap? | Use Z-spread or market ASW for cross-bond comparison |
| 9 | Define $P_{\text{Libor}}(0,T)$. | $(c/f) \sum Z + Z(T)$—Libor-discounted value of bond cashflows |
| 10 | How does market ASW spread $A^*$ relate to $A$? | $A^* = A/P$ |
| 11 | What is the CDS-bond basis? | CDS spread minus ASW spread |
| 12 | What does a very negative CDS-bond basis indicate? | Funding stress; bonds cheap but arbitrage can't close the gap |
| 13 | What happens to swap obligations if the bond defaults? | Bond coupons cease but fixed swap payments remain due |
| 14 | Define ZVS. | Fixed spread adjustment to Libor discount rate that reprices the bond |
| 15 | What is DV01? | $-\Delta P / (10{,}000 \cdot \Delta y)$—dollar value of 1 bp |
| 16 | What five components make up ASW P&L? | Rates, spread, carry, rolldown, funding |
| 17 | How does repo affect ASW carry? | Higher repo = lower carry; "special" repo = higher carry |
| 18 | Why use ASW for longer maturities? | Measures value relative to curves less contaminated by security-specific effects |
| 19 | What does positive ASW indicate? | Bond is cheap vs. the Libor/swap discount curve |
| 20 | Can ASW spreads be negative? | Yes, for issuers perceived better than AA banks |

---

## Mini Problem Set

**Questions 1–8 include solution sketches.**

**1.** Swap spread: $S_7 = 3.55\%$, $y_7^{\text{OTR}} = 3.40\%$. Compute swap spread in bp.

*Solution:* $(3.55 - 3.40)\% = 0.15\% = 15$ bp.

**2.** Using Q1, fitted gov yield is 3.46%. Recompute swap spread and the difference vs. OTR.

*Solution:* $3.55 - 3.46 = 0.09\% = 9$ bp; difference is $15 - 9 = 6$ bp lower.

**3.** ASW inputs: Clean price $P_c = 101.20$, AI $= 0.30$ (per 100). What full price do you use?

*Solution:* $P = 101.20 + 0.30 = 101.50 \Rightarrow 1.0150$ per par 1.

**4.** Compute PV01: Semiannual schedule with $Z = \{0.99, 0.975, 0.96, 0.945\}$, $\Delta = 0.5$.

*Solution:* $\sum Z = 3.87$; PV01 $= 0.5 \times 3.87 = 1.935$.

**5.** Compute $P_{\text{Libor}}$: Coupon $c = 4\%$, $f = 2$, same $Z$, maturity at last date with $Z(T) = 0.945$.

*Solution:* $P_{\text{Libor}} = 0.02 \times 3.87 + 0.945 = 0.0774 + 0.945 = 1.0224$.

**6.** Solve ASW: Use Q3–Q5 with $P = 1.0150$. Compute $A$ in bp.

*Solution:* $A = (1.0224 - 1.0150)/1.935 = 0.0074/1.935 = 0.00382 \approx 38.2$ bp.

**7.** Market ASW: Use Q6 and $P = 1.0150$.

*Solution:* $A^* = A/P \approx 0.00382/1.0150 = 0.00376 \approx 37.6$ bp.

**8.** A high-coupon bond (8%) trades at 115, with par ASW of 180 bp. A low-coupon bond (3%) from the same issuer trades at 92, with par ASW of 90 bp. Which looks cheaper on par ASW? Is this comparison valid?

*Solution:* The 8% bond looks cheaper (180 bp vs. 90 bp). However, this is the premium bond trap—the 8% bond's wide par ASW includes premium amortization. Use Z-spread to compare validly.

**9.** Name two reasons why the CDS-bond basis was extremely negative during 2008-2009.

**10.** Explain why pension fund hedging demand pushes swap spreads more negative.

**11.** A trader says "swap spreads at -20 bp are free money—Treasuries yield more than swaps!" Explain why this view is flawed.

**12.** Describe how repo rate spikes at quarter-end affect ASW carry.

---

## Source Map

### (A) Book-Verified Facts

| Fact | Source |
|------|--------|
| Swap spread definition $\text{SS}_T = S_T - y_T^{gov}$ with on-the-run convention | Tuckman Ch 18 |
| Swap rates reflect rolling bank credit; "continually refreshed rate" | Tuckman Ch 18, Hull RM Ch 9 |
| On-the-run yields embed liquidity premiums and special financing | Tuckman Ch 15, 18 |
| Historical swap spread dynamics (1990s banking recovery, late-1990s Treasury scarcity) | Tuckman Ch 18 |
| ASW defined as spread such that discounting bond cashflows by swap+spread gives price | Tuckman Ch 18 |
| Par asset swap formula $A = (P_{\text{Libor}} - P)/\text{PV01}$ | O'Kane Ch 4 |
| ASW is equivalent to long defaultable bond, short risk-free bond with same coupons | O'Kane Ch 4 |
| Market asset swap $A^* = A/P$ with notional = full price | O'Kane Ch 4 |
| Market asset swap reverses timing and direction of counterparty exposure | O'Kane Ch 4 |
| Fixed swap payments are non-contingent (continue even if bond defaults) | O'Kane Ch 4 |
| ZVS as fixed spread adjustment to Libor discount rate | O'Kane Ch 4 |
| ZVS preferred over yield because it accounts for term structure | O'Kane Ch 4 |
| Bond default stops coupons but swap payments remain due | Tuckman Ch 18, O'Kane Ch 4 |
| Asset swaps give direct estimates of excess yields over LIBOR/swap rates | Hull RM Ch 19 |
| CDS-bond basis definition: CDS spread minus asset swap spread | Hull RM Ch 19, O'Kane Ch 5 |
| CDS-bond basis typically positive pre-2007, very negative during crisis | Hull RM Ch 19 |
| Six factors affecting CDS-bond basis | Hull RM Ch 19 |
| Multi-curve separation (OIS discounting vs. Libor projection) | Andersen & Piterbarg Vol 1, Hull RM Ch 9 |
| DV01 definition | Tuckman Ch 5 |
| Only with flat swap curve does ASW = bond yield − swap rate | Tuckman Ch 18 footnote |

### (B) Claude-Extended Content (Practitioner Knowledge)

| Content | Context |
|---------|---------|
| Three drivers of negative swap spreads (Treasury supply, SLR, pension/LDI) | Extended from Tuckman's supply discussion; SLR and LDI are post-publication developments well-known to practitioners |
| Premium bond trap explanation and worked example | Derived from O'Kane's market ASW discussion; trap phenomenon is practitioner knowledge |
| P&L attribution framework (5 components) | Extended from Tuckman's carry discussion; standard desk practice |
| Desk Reality boxes | Practitioner color based on market experience |
| Balance sheet constraints preventing arbitrage | Extended from crisis discussion in Hull RM |

### (C) Reasoned Inference (Derived from A or B)

| Inference | Derivation Logic |
|-----------|------------------|
| Benchmark choice can change swap spreads by multiple bp | Direct application of definition with different $y_T^{gov}$ |
| ASW and ZVS differ due to curve shape, conventions, multi-curve | Structural difference in definitions |
| ASW spread sensitivity $\partial V / \partial A = \text{PV01}$ | Differentiation of O'Kane's swap value formula |
| DV01 mismatch in bond-vs-swap when bond yield $\neq$ swap rate | Follows from Tuckman's discussion |
| Negative ASW possible for high-quality issuers | Follows from formula when $P > P_{\text{Libor}}$ |
| Premium bond par ASW includes price/par amortization | Follows from par ASW structure |

### (D) Flagged Uncertainties

| Item | Note |
|------|------|
| Full multi-curve ASW computation under OIS | **I'm not sure** — requires specifying projection curve, CSA terms, instrument set; sources confirm concept but don't provide complete desk conventions |
| Exact magnitude of LDI flows and timing | **I'm not sure** — the effect is well-known but quantification varies by market condition |
| Precise SLR capital calculation for swap spread arbitrage | **I'm not sure** — depends on bank-specific factors and regulatory interpretation |

---

*Chapter 27 complete.*
