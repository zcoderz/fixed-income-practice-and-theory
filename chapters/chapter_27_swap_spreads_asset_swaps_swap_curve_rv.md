# Chapter 27: Swap Spreads, Asset Swaps, and Swap-Curve Relative Value

---

## Introduction

The 10-year swap spread is 30 basis points. What does that actually tell you?

At first glance, the answer seems obvious: swap rates are 30 bp above Treasury yields of the same maturity, reflecting the credit risk embedded in bank funding rates versus the "risk-free" government curve. But this simple interpretation masks a web of complications that make swap spreads one of the most misunderstood quantities in fixed income. The Treasury yield you subtract may be distorted by on-the-run liquidity premiums and repo specialness. The swap rate reflects rolling short-term bank credit—not a static credit spread. And the spread itself conflates credit, supply/demand dynamics, and market microstructure in ways that can mislead the unwary trader.

This chapter unpacks the mechanics and interpretation of swap spreads and their close relative, the asset swap spread. Where swap spreads compare swap rates to government yields directly, asset swap spreads anchor individual bonds to the swap curve—providing a cleaner measure of a bond's value relative to bank-credit-based funding. Understanding the distinction matters because the two measures answer different questions and can move in opposite directions.

The chapter proceeds as follows. Section 27.1 defines swap spreads precisely and examines why benchmark choice matters so much. Section 27.2 introduces asset swaps as both a spread measure and a trading structure, developing the key pricing formulas from O'Kane's framework. Section 27.3 contrasts the zero-volatility spread (ZVS) with asset swap spreads, explaining when they diverge. Section 27.4 covers risk measurement and P&L attribution for asset swap positions. Section 27.5 develops the swap-curve relative value framework that traders use to identify cheap and rich bonds. Section 27.6 addresses multi-curve considerations that arise in modern practice.

The reader should come away understanding: (1) why swap spreads are not "pure credit spreads," (2) how to compute and interpret asset swap spreads using both Tuckman's and O'Kane's approaches, (3) the structural differences between ASW and ZVS, and (4) how desks use these measures for relative value trading.

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

## 27.2 Asset Swaps: Structure, Pricing, and Conventions

### 27.2.1 Motivation: Anchoring Bonds to the Swap Curve

For short-term securities, TED spreads (based on Eurodollar futures rates) historically provided a convenient measure of value relative to LIBOR. But as Tuckman explains, "For longer-term bonds, market participants rely on asset swap spreads." The asset swap spread of a bond measures its value relative to the swap curve rather than to Treasuries, avoiding the idiosyncratic effects that contaminate Treasury-based spreads.

Tuckman defines the asset swap spread as "the spread such that discounting the bond's cash flows by swap rates plus that spread gives the bond price." This definition frames the asset swap spread as a *spread measure*—a way to quantify where a bond trades relative to the swap curve.

O'Kane provides a complementary perspective, framing the asset swap as a *contract structure* that combines a bond with an interest rate swap. He notes that since its inception in the early 1980s, the asset swap "has become an extremely important product for credit investors" and is "considered by some to be the first credit derivative." Hull's Risk Management text adds that asset swaps provide "direct estimates of the excess of bond yields over LIBOR/swap rates."

> **Concept: The Synthetic Bond**
>
> Think of an Asset Swap (ASW) as a "Currency Converter for Credit."
>
> *   **The Problem**: You want to buy Apple's credit risk, but Apple only issues Fixed Rate bonds. You are a Floating Rate investor (like a bank). You speak "Floating," Apple speaks "Fixed."
> *   **The Solution**: You buy the bond, but you wrap it in an Interest Rate Swap.
>     *   **The Swap**: Takes the Fixed Coupons you receive from Apple and converts them into Floating Rate payments (LIBOR/SOFR).
>     *   **The Result**: You now own a "Synthetic Floating Rate Note" issued by Apple.
>     *   **The Spread**: The extra spread you get above LIBOR/SOFR is the "Asset Swap Spread." It is the pure price of Apple's credit, stripped of interest rate duration.

### 27.2.2 Asset Swap Mechanics: The Par Asset Swap

In a par asset swap, an investor combines a bond purchase with an interest rate swap structured so that the combined package costs par. O'Kane describes the mechanics in detail:

**On settlement.** The asset swap buyer receives a fixed-rate bond with full price $P$ and pays par (i.e., 100% of face value). If $P \neq 1$, there is an upfront payment of $(1 - P)$ from the buyer (if the bond is at a discount) or to the buyer (if at a premium).

**Interest rate swap.** The buyer enters a payer swap with fixed leg payments matching the bond coupon schedule. On the floating leg, the buyer receives LIBOR plus the asset swap spread $A$.

**Cash flow netting.** The bond coupons received offset the fixed swap payments, leaving the buyer with a net position of receiving LIBOR + $A$ minus the cost of financing the bond in repo.

O'Kane emphasizes a critical detail: "It is essential to realise that these swap fixed leg payments are non-contingent, meaning that they are scheduled to be paid even if the bond defaults." This asymmetry—bond coupons cease on default but swap payments continue—is a key risk feature of asset swaps that distinguishes them from simple floating rate notes.

The implications are significant. If the bond defaults immediately after the asset swap begins, the investor loses both the bond (now worth only recovery $R$) and continues to owe fixed payments on the swap. This creates exposure beyond the simple credit risk of the bond itself.

> **Visual: The Package Deal**
>
> *   **Left Side (Physical)**: You Buy Bond. Pays Fixed Coupon $C$.
> *   **Middle (Swap)**: You Pay Fixed $C$ to Dealer. You Receive Floating (LIBOR + Spread).
> *   **Net Result**: The Fixed Coupons cancel out ($+C - C = 0$). You are left with just the Floating Stream (LIBOR + Spread).
>
> If the Bond Defaults, the Left Side disappears, but the Middle Side (Swap) implies you *still* owe Fixed Coupons. This is why Asset Swaps are risky!

### 27.2.3 The Par Asset Swap Spread Formula

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

Negative asset swap spreads can occur for very high-quality issuers whose credit is perceived as better than the AA bank credit embedded in LIBOR—or for bonds with embedded options that create additional value.

> **Why Negative Spreads?**
>
> How can a risky bond trade *below* the Swap Rate (Negative Spread)?
>
> 1.  **Treasury Scarcity**: When "safe" assets are rare, people pay a huge premium for them. If Treasuries are incredibly expensive (yields are tiny), their yield might fall *below* the swap rate.
> 2.  **The "Better than Banks" Effect**: In times of banking crisis (e.g., 2008), the Swap Rate (which reflects Bank Credit) spikes. A sovereign government (like the US or Germany) is safer than a bank. So the government yield trades *below* the bank swap rate.
>
> If you are "richer" than a bank, your spread is negative.

### 27.2.4 Worked Example: Computing the Par Asset Swap Spread

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

### 27.2.5 The Market Asset Swap: An Alternative Structure

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

### 27.2.6 Mark-to-Market of an Asset Swap

O'Kane shows that the mark-to-market value of a par asset swap at time $t$ is:

$$\text{MTM}(t) = (A(0) - A(t)) \cdot \text{PV01}(t,T)$$

where $A(t)$ is the current asset swap spread for the bond. This formula follows from the fact that unwinding an asset swap involves entering an offsetting position at the current spread. The fixed legs and Libor payments cancel, leaving only the difference in spreads times the remaining PV01.

**Worked example.** Using O'Kane's numbers: if the asset swap was entered at $A(0) = 323.9$ bp on $10 million notional, and one year later the spread has tightened to $A(t) = 284$ bp with remaining PV01 of 3.622, the mark-to-market is:

$$\text{MTM} = (323.9 - 284.0) \text{ bp} \times 3.622 \times \$10\text{mm} = 39.9 \times 0.0001 \times 3.622 \times 10{,}000{,}000 = \$144{,}518$$

The position has profited because the credit quality of the issuer improved (spread tightened), as O'Kane notes: "The credit quality of the issuer has improved, as shown by the increase in the bond price, and the position has made money."

### 27.2.7 Asset Swaps in Tuckman's Framework

Tuckman presents asset swaps through a concrete trading example. Consider an investor who believes the FNMA 6.25s of May 15, 2029 are cheap relative to swaps at an asset swap spread of 15 bp. The investor executes:

1. Buy $100 million face of the bond at par
2. Finance the position in repo
3. Enter a swap paying 6.25% fixed and receiving LIBOR + 15 bp

The result is that "the coupon payments from the bond cancel the fixed payments to the swap," leaving the investor receiving "LIBOR plus 15 basis points minus the repo rate" so long as the agency does not default.

Tuckman emphasizes: "the asset swap spread is a measure of the credit risk of the bond relative to the swap curve while LIBOR minus repo is a measure of the credit risk of the swap curve." An investor who enters the asset swap earns the full credit spread of the bond—but loses everything if the bond defaults, since the swap obligations remain.

**Premium bond adjustment.** When the bond trades at a premium, Tuckman shows that the swap must include an upfront payment. In his example, when the FNMA bond trades at 102.683, the swap is structured so the investor receives an upfront payment of $2.683 million, which is applied to the bond purchase. This leaves $100 million financed in repo, maintaining comparability of the floating rate receipts.

Tuckman notes a subtle point about default risk: "When the bond is selling at a premium, an event of default costs more. Put another way, the 6.25% coupon stream at risk is worth more in the second case, when the swap rate is 5.90%, than in the first case, when the swap rate is 6.10%."

---

## 27.3 Zero-Volatility Spread: Definition and Relation to ASW

### 27.3.1 ZVS Definition

The zero-volatility spread (ZVS), also called the Z-spread, takes a different approach from asset swap spreads. O'Kane defines the ZVS as "the fixed spread adjustment to the Libor discount rate which reprices the bond."

In discrete compounding form:

$$P = \frac{c}{f} \sum_{n=1}^{N} \frac{1}{(1 + (r(0,t_n) + \theta)/f)^n} + \frac{1}{(1 + (r(0,t_N) + \theta)/f)^N}$$

Or in continuous compounding form:

$$P = \frac{c}{f} \sum_{n=1}^{N} Z(0,t_n) e^{-\theta t_n} + Z(0,T) e^{-\theta T}$$

where $\theta$ is the ZVS to be solved for numerically.

O'Kane prefers the term "zero volatility spread" to "option-adjusted spread" (OAS) for bonds without embedded options, noting that "the concept of the OAS has its origins in the callable bond market" and using the term for plain fixed-coupon bonds "can be confusing." Some practitioners also hedge using the ZVS rather than the yield because, as O'Kane notes, "the ZVS is preferred because it takes into account the term structure of the interest rate curve."

### 27.3.2 Conceptual Difference: ZVS vs. ASW

Although ASW and ZVS both measure a bond's spread to the swap curve, they are conceptually different:

| Aspect | ZVS | ASW |
|--------|-----|-----|
| **Mechanics** | Discounting spread: add $\theta$ to all discount rates | Swap-structure spread: spread on floating leg in a package |
| **What it values** | PV of bond cashflows with shifted discount curve | Floating leg receipts in swap-plus-bond package |
| **Solved from** | $P = \sum \text{DF}_{\text{shifted}} \times \text{CF}$ | $V(0) + P = 1$ in par asset swap structure |
| **Numerical method** | Root-finding (iterate $\theta$) | Direct formula |

For simple fixed-rate bonds under single-curve assumptions with a flat curve, the two measures are close. Tuckman notes that "only in the very special case of a flat swap rate curve does the asset swap spread equal the difference between the bond yield and the swap rate."

**When the measures diverge.** The ASW and ZVS can differ significantly when:

1. **Curve shape is non-flat.** ZVS shifts all discount rates uniformly by the same spread. ASW is determined by a PV01-weighted average that depends on the structure of the swap.

2. **Conventions differ.** Clean vs. dirty price inputs, par/par vs. market/market structures can create discrepancies.

3. **Multi-curve discounting.** OIS discounting with separate projection curves affects ASW and ZVS differently because ASW involves an actual swap contract.

4. **Funding/repo effects.** Realized P&L of an asset swap depends on repo rates, which ZVS ignores entirely.

### 27.3.3 Worked Example: ASW vs. ZVS Comparison

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

The small difference (0.5 bp) arises from the structural distinction: ZVS applies a uniform shift to all discount rates, while ASW is determined through the PV01-weighted average implicit in the swap structure. For practical purposes, the measures are often close enough to be treated as equivalent for plain bonds.

---

## 27.4 Risk Measurement and P&L Attribution

### 27.4.1 DV01 and Hedge Ratios

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

### 27.4.2 Spread Sensitivity

The sensitivity of the swap value to changes in the asset swap spread follows from differentiation of O'Kane's formula:

$$\frac{\partial V}{\partial A} = \text{PV01}$$

This means a 1 bp change in $A$ changes the swap value by approximately PV01 per unit notional. For a $100 million position with PV01 of 2.763 discounted years, a 1 bp change in ASW represents approximately:

$$\Delta V = 0.0001 \times 2.763 \times 100{,}000{,}000 = \$27{,}630$$

### 27.4.3 P&L Decomposition for Asset Swap Positions

An asset swap position's P&L can be decomposed into three main components:

**Rates P&L.** From parallel or non-parallel rate moves, captured through DV01 and key-rate exposures. If the position is DV01-hedged, this component should be small for parallel shifts but can be significant for curve twists.

**Spread P&L.** From changes in the asset swap spread, proportional to PV01. This is typically the intended exposure in an asset swap trade.

**Funding P&L.** From the difference between LIBOR receipts and repo financing costs. Tuckman notes that in his FNMA example, the investor's periodic receipts are "LIBOR plus 15 basis points minus the repo rate." The funding component is often overlooked but can materially affect realized returns, particularly in stressed markets when repo rates diverge from LIBOR.

### 27.4.4 Worked Example: P&L Scenarios for an ASW Position

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

As Tuckman emphasizes, even positions framed as "collect the spread" can experience significant mark-to-market volatility from spread changes, potentially forcing position reductions at the worst time.

---

## 27.5 Swap-Curve Relative Value Framework

### 27.5.1 The RV Trader's Perspective

Swap-curve relative value is not a single formula but a *practice*—the comparison of bond value to the swap curve rather than to Treasuries. Tuckman motivates this approach: market participants use asset swap spreads and similar measures for longer-term bonds "to measure value relative to curves not contaminated by individual security effects."

The key questions in swap-curve RV are:
- Is this bond cheap or rich relative to the swap curve?
- How does its ASW compare to similar bonds or its own history?
- What would need to happen for the ASW to normalize?

### 27.5.2 Rich/Cheap Analysis

A bond is considered "cheap on ASW" when its asset swap spread is wide relative to:
- Historical levels for that issuer
- Comparable bonds in the same sector
- The spread implied by CDS levels (the "basis")

Conversely, a bond is "rich on ASW" when its spread is tight relative to these benchmarks.

O'Kane provides the intuition: "The asset swap spread is therefore a measure of the credit quality of the fixed rate bond. If the bond issuer has the credit quality of the AA commercial banking sector, then it will probably have a price close to $P_{\text{Libor}}$ and the asset swap spread will be close to zero."

For investment-grade issuers, ASW spreads are typically positive but modest. For high-yield issuers, ASW spreads can be hundreds of basis points. Negative ASW spreads signal that the market views the issuer as *better* than the AA banking sector—rare but not impossible for sovereigns or supranationals.

### 27.5.3 Tuckman's Spread-of-Spreads Trade

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

### 27.5.4 What Drives ASW Changes

Asset swap spreads can widen or tighten for several reasons:

| Driver | Effect on ASW |
|--------|---------------|
| Issuer credit deterioration | Widens |
| General flight-to-quality | Widens (for non-government issuers) |
| Bond-specific supply pressure | Widens |
| Issuer credit improvement | Tightens |
| Strong bid for the sector | Tightens |
| Swap curve steepening/flattening | Can affect ASW through PV01 changes |

**The CDS-Bond Basis.** The difference between a bond's ASW spread and its CDS spread provides additional context. Hull's Risk Management text defines the CDS-bond basis as "the excess of the CDS spread over the asset swap spread." Prior to 2007, the basis was typically positive (CDS spreads slightly higher than ASW), but "during the credit crisis, the basis was at times very negative" due to liquidity constraints that prevented arbitrage.

A very negative basis (CDS tight relative to ASW) can signal:
- Bonds are under supply pressure
- Funding is expensive for bond positions
- Arbitrageurs are unable to exploit the apparent mispricing

---

## 27.6 Multi-Curve Considerations

### 27.6.1 The Shift to OIS Discounting

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

### 27.6.2 Sensitivity to Discounting Assumptions

Changing from legacy single-curve Libor discounting to OIS discounting can materially change ASW levels. As a sensitivity experiment, consider the bond from Section 27.3.3 with slightly higher OIS discount factors (reflecting OIS below LIBOR):

| Curve | Sum of $Z$ | PV01 | $P_{\text{disc}}$ | ASW |
|-------|-----------|------|-------------------|-----|
| Libor | 5.674 | 2.837 | 1.04685 | $-11.1$ bp |
| OIS (toy) | 5.688 | 2.844 | 1.0512 | $+4.2$ bp |

The ASW flips from negative to positive purely from the discounting change. This illustrates why stating the discounting convention is essential for any precise ASW calculation.

**Practical implication.** When comparing ASW levels across different sources (Bloomberg, Reuters, internal systems), verify that the discounting methodology is consistent. A 5 bp difference might reflect methodology, not relative value.

---

## 27.7 Worked Examples: Consolidated

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

### Common Pitfalls

| Pitfall | Description |
|---------|-------------|
| Mixing swap spread with ASW | Swap spread is (swap − gov); ASW is (bond vs. swap curve)—not identical |
| ZVS vs. ASW confusion | Different constructions (discounting spread vs. swap-package spread) |
| Clean/dirty mismatch | Using clean price where full price is required (or vice versa) |
| Ignoring default terms | Bond default does not extinguish swap fixed payments |
| Forgetting basis and funding | Treasury specialness affects swap spreads; repo levels affect realized carry |
| Compounding mismatches | Semiannual vs. money-market conventions across curves |

### Verification Tests

**Repricing check.** For a par asset swap, verify $V(0) + P = 1$.

**Sign check.** From $A = (P_{\text{Libor}} - P)/\text{PV01}$, if bond price $P$ falls (bond cheapens), $A$ should rise (widen).

**Unit check.** DV01 is per-100 price points; convert to dollars by $\text{Notional} \times \text{DV01} / 100$.

---

## Summary

This chapter has developed the framework for understanding swap spreads, asset swaps, and swap-curve relative value:

1. **Swap spreads** ($\text{SS}_T = S_T - y_T^{gov}$) compare swap rates to government yields but are contaminated by on-the-run liquidity and specialness effects.

2. **Swap spreads are not pure credit spreads**—they reflect rolling bank credit (a "continually refreshed rate"), Treasury supply/demand, and market microstructure jointly.

3. **Asset swap spreads** anchor bonds to the swap curve, measuring value relative to bank-credit-based funding. The formula $A = (P_{\text{Libor}} - P)/\text{PV01}$ provides the par ASW spread.

4. **Market asset swaps** use notional equal to bond price, with $A^* = A/P$, reversing the timing and direction of counterparty exposure.

5. **ZVS** is a discounting spread that differs structurally from ASW, though both measure bond value relative to swaps.

6. **ASW trades carry rates risk** (DV01 mismatch) and **spread risk** (ASW changes); funding and repo affect realized returns.

7. **Default asymmetry** is crucial: bond defaults stop coupons but swap obligations continue, creating asymmetric exposure.

8. **Benchmark choice** can change swap spread readings by multiple basis points, complicating time series analysis.

9. **Multi-curve frameworks** (OIS discounting) require careful specification of discount and projection curves.

10. **Spread-of-spreads trades** isolate relative value between two securities by DV01-matching each leg.

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| Swap spread | $S_T - y_T^{gov}$ | Broad swaps vs. gov indicator, but benchmark-sensitive |
| Asset swap spread | Spread on floating leg making bond+swap = par | Bond valuation vs. swap curve |
| $P_{\text{Libor}}$ | PV of bond cashflows on swap curve | Reference for cheap/rich determination |
| PV01 | Discounted accrual sum | Denominates spread formulas and sensitivities |
| ZVS | Discounting spread over Libor curve | Alternative spread measure with different mechanics |
| Market ASW | $A^* = A/P$ | Reduces counterparty risk for off-par bonds |
| Rolling credit | Swap reflects continually refreshed short-term credit | Why swap ≠ term bank bond yield |
| CDS-bond basis | CDS spread − ASW spread | Indicator of relative value and market stress |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | Define swap spread at maturity $T$. | $\text{SS}_T = S_T - y_T^{gov}$, usually vs. on-the-run gov yield |
| 2 | Why is on-the-run benchmarking problematic? | On-the-run yields include liquidity premiums and special financing effects |
| 3 | Give two desk alternatives to on-the-run swap spreads. | Fitted/off-the-run gov curve yields or adjust OTR yields for financing/liquidity |
| 4 | What does "continually refreshed rate" mean for swaps? | Swap rates reflect rolling short-term bank credit, not static term credit |
| 5 | Tuckman's definition of asset swap spread? | Spread such that discounting bond cashflows by swap rates plus spread gives bond price |
| 6 | What bond price goes into ASW formulas: clean or dirty? | Full/dirty price $P = P_c + \text{AI}$ |
| 7 | What is $\text{PV01}(0,T)$ in O'Kane's ASW formula? | $\sum Z(0,t_n) \Delta(t_{n-1}, t_n)$—discounted accrual sum |
| 8 | Write the par ASW spread formula. | $A = (P_{\text{Libor}} - P)/\text{PV01}$ |
| 9 | Define $P_{\text{Libor}}(0,T)$. | $(c/f) \sum Z + Z(T)$—Libor-discounted value of bond cashflows |
| 10 | How does market ASW spread $A^*$ relate to $A$? | $A^* = A/P$ |
| 11 | Why introduce market asset swaps? | To avoid counterparty risk when exchanging off-par bonds for par |
| 12 | What happens to swap obligations if the bond defaults? | Bond coupons cease but fixed swap payments remain due |
| 13 | What limits losses in a standard swap under counterparty default? | No principal is exchanged; loss is limited to positive replacement value |
| 14 | Define ZVS. | Fixed spread adjustment to Libor discount rate that reprices the bond |
| 15 | Give the continuous ZVS pricing equation. | $P = (c/f) \sum Z e^{-\theta t} + Z(T) e^{-\theta T}$ |
| 16 | Why prefer "ZVS" over "OAS" for plain bonds? | OAS originated in callable bonds; ZVS avoids confusion for plain bonds |
| 17 | What is DV01? | $-\Delta P / (10{,}000 \cdot \Delta y)$—dollar value of 1 bp |
| 18 | How size a DV01 hedge between two instruments? | Notional ratio proportional to DV01s |
| 19 | Two major P&L drivers of asset swap trades? | Rate moves (DV01 mismatch) and ASW spread changes |
| 20 | Why use ASW for longer maturities? | Measures value relative to curves less contaminated by security-specific effects |
| 21 | What does positive ASW indicate in O'Kane's formula? | Bond is cheap vs. the Libor/swap discount curve |
| 22 | Can ASW spreads be negative? | Yes, for issuers perceived better than AA banks |
| 23 | Why might swap spreads rise even if bank credit improves? | Treasury scarcity can lower Treasury yields, widening spreads |
| 24 | Key difference between swap spread and ASW? | Swap spread is swaps vs. gov; ASW is bond vs. swap curve |
| 25 | In a par asset swap, what upfront payment occurs if bond trades at 95? | Buyer pays 100, receives bond worth 95—net $5 payment by buyer |
| 26 | Why can swap-hedging corporate debt leave basis risk? | Swap rates reflect banking credit, not issuer-specific spreads |
| 27 | What does $P = P_c + \text{AI}$ protect against? | Clean/dirty mismatches that misstate PV and spread |
| 28 | What's the ASW spread sensitivity? | $\partial V / \partial A = \text{PV01}$ |
| 29 | Why compute swap spreads vs. fitted gov yields? | To reduce distortion from on-the-run liquidity/financing |
| 30 | Why does discounting choice (OIS vs. legacy) matter for ASW? | Different discount curves change PV and implied spreads |

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

**8.** ZVS sign intuition: If $P > P_{\text{Libor}}$, what is the sign of $\theta$?

*Solution:* Need higher PV $\Rightarrow \theta < 0$ (negative spread increases PV).

**9.** Compare swap spread vs. ASW: give one scenario where swap spread widens but ASW for a given corporate bond tightens.

**10.** Describe how repo specialness in the benchmark Treasury can distort a quoted swap spread.

**11.** Using Tuckman's caution, propose a desk procedure to compute "adjusted swap spreads" and list its weaknesses.

**12.** Give two reasons ASW and ZVS can diverge for the same bond even without embedded options.

**13.** Explain why an asset swap is not a perfect "credit-only" trade even if DV01-matched.

**14.** Suppose OIS discounting is adopted. List the minimum additional conventions/data needed for rigorous ASW.

**15.** A bond's ASW spread widens 25 bp with yield curve unchanged. Explain the likely bond price direction and P&L of a long-ASW position.

**16.** Construct a DV01-matched bond vs. swap hedge using DV01s of 0.030 and 0.045 (per 100). What swap notional per $100mm bond?

---

## Source Map

### (A) Verified Facts (Source-Backed)

| Fact | Source |
|------|--------|
| Swap spread definition $\text{SS}_T = S_T - y_T^{gov}$ with on-the-run convention | Tuckman Ch 18 |
| Swap rates reflect rolling bank credit; "continually refreshed rate" | Tuckman Ch 18, Hull RM Ch 9 |
| On-the-run yields embed liquidity premiums and special financing | Tuckman Ch 15, 18 |
| ASW defined as spread such that discounting bond cashflows by swap+spread gives price | Tuckman Ch 18 |
| Par asset swap formula $A = (P_{\text{Libor}} - P)/\text{PV01}$ | O'Kane Ch 4 |
| ASW is equivalent to long defaultable bond, short risk-free bond with same coupons | O'Kane Ch 4 |
| Market asset swap $A^* = A/P$ with notional = full price | O'Kane Ch 4 |
| Market asset swap reverses timing and direction of counterparty exposure | O'Kane Ch 4 |
| ZVS as fixed spread adjustment to Libor discount rate | O'Kane Ch 4 |
| ZVS preferred over yield because it accounts for term structure | O'Kane Ch 4 |
| Bond default stops coupons but swap payments remain due | Tuckman Ch 18, O'Kane Ch 4 |
| Asset swaps give direct estimates of excess yields over LIBOR/swap rates | Hull RM Ch 19 |
| Historical swap spread dynamics (1990s banking recovery, late-1990s Treasury scarcity) | Tuckman Ch 18 |
| CDS-bond basis definition: CDS spread minus asset swap spread | Hull RM Ch 19, O'Kane Ch 5 |
| CDS-bond basis typically positive pre-2007, very negative during crisis | Hull RM Ch 19 |
| Multi-curve separation (OIS discounting vs. Libor projection) | Andersen & Piterbarg Vol 1, Hull RM Ch 9 |
| DV01 definition | Tuckman Ch 5 |
| Only with flat swap curve does ASW = bond yield − swap rate | Tuckman Ch 18 footnote |

### (B) Reasoned Inference (Derived from A)

| Inference | Derivation Logic |
|-----------|------------------|
| Benchmark choice can change swap spreads by multiple bp | Direct application of definition with different $y_T^{gov}$ |
| ASW and ZVS differ due to curve shape, conventions, multi-curve | Structural difference in definitions |
| ASW spread sensitivity $\partial V / \partial A = \text{PV01}$ | Differentiation of O'Kane's swap value formula |
| DV01 mismatch in bond-vs-swap when bond yield $\neq$ swap rate | Follows from Tuckman's discussion |
| Negative ASW possible for high-quality issuers | Follows from formula when $P > P_{\text{Libor}}$ |

### (C) Flagged Uncertainties

| Item | Note |
|------|------|
| Full multi-curve ASW computation under OIS | **I'm not sure** — requires specifying projection curve, CSA terms, instrument set; sources confirm concept but don't provide complete desk conventions |
| Desk terminology for "G-spread" vs "I-spread" | **I'm not sure** — definitions vary somewhat across practitioners and systems |
| Exact canonical swap-spread-trade sign convention | **I'm not sure** — depends on market, implementation, and desk convention |

---

*Chapter 27 complete.*
