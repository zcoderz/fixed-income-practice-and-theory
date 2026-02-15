# Chapter 27: Swap Spreads, Asset Swaps, and Swap-Curve Relative Value

---

## Introduction

Suppose you see a screen quote: the 10-year swap spread is \(-20\) bp. The U.S. government—often treated as the "risk-free" benchmark—yields *more* than the par swap rate. How is this possible?

Negative swap spreads feel like a "credit paradox" if you interpret the swap spread as a pure bank-credit spread over Treasuries. If that story were literally true, swap spreads "shouldn't" be negative. In practice, the swap spread is a relative-value quote that mixes multiple forces: benchmark choice (on-the-run liquidity and repo specialness), Treasury supply/demand and financing conditions, and the fact that the “Treasury yield” and “swap rate” embed different premia and conventions.

Understanding this puzzle requires unpacking what swap spreads actually measure. At first glance, the answer seems obvious: swap rates are above (or below) Treasury yields of the same maturity, reflecting credit and liquidity differences. But this simple interpretation masks a web of complications. The Treasury yield you subtract may be distorted by on-the-run liquidity premiums and repo specialness. The swap rate reflects rolling short-term bank credit—not a static credit spread. And the spread itself can mix credit, Treasury-side distortions, and funding/hedging mechanics in ways that can mislead the unwary trader.

This chapter unpacks the mechanics and interpretation of swap spreads and their close relative, the asset swap spread. Where swap spreads compare swap rates to government yields directly, asset swap spreads anchor individual bonds to the swap curve—providing a cleaner measure of a bond's value relative to bank-credit-based funding. Understanding the distinction matters because the two measures answer different questions and can move in opposite directions.

Prerequisites: [Chapter 25 — Interest Rate Swaps — Mechanics and Valuation](chapters/chapter_25_interest_rate_swaps_mechanics_valuation.md); [Chapter 26 — Swap PV01, DV01, and Hedging with Swaps](chapters/chapter_26_swap_pv01_dv01_hedging.md); [Chapter 9 — Repo and Secured Funding](chapters/chapter_09_repo_funding_engine.md) (optional: [Chapter 8 — Spread Measures](chapters/chapter_08_spreads_101.md)).

Follow-on: [Chapter 28 — Basis Trades in Rates](chapters/chapter_28_basis_trades.md).

## Learning Objectives
- Translate a swap-spread quote into the underlying objects being compared (par swap rate vs a chosen government benchmark yield).
- Explain why benchmark choice (on-the-run vs fitted/off-the-run) can move a quoted swap spread by several bp without any change in “credit”.
- Compute and interpret a par asset swap spread and the market asset swap spread, stating clean vs dirty inputs and units.
- Explain the “premium bond trap” and choose a spread measure (ASW vs ZVS/Z-spread vs CDS) consistent with the question you’re asking.
- State the risk of an asset-swap position with an explicit bump object, bump size (1bp \(=10^{-4}\)), units, and sign convention.

The chapter proceeds as follows. Section 27.1 defines swap spreads precisely and examines why benchmark choice matters. Section 27.2 explains why swap spreads can be negative and why negative spreads can persist. Section 27.3 introduces asset swaps as both a spread measure and a trading structure. Section 27.4 addresses the "premium bond trap"—a critical gotcha in asset swap analysis. Section 27.5 contrasts the zero-volatility spread (ZVS) with asset swap spreads. Section 27.6 develops the CDS-bond basis framework that links cash and derivative credit markets. Section 27.7 covers risk measurement, P&L attribution, and funding effects. Section 27.8 presents the swap-curve relative value framework. Section 27.9 addresses multi-curve considerations.

The reader should come away understanding: (1) what swap spreads measure and how benchmark choice distorts them, (2) how to compute and interpret ASW and ZVS-type spread measures for a bond, (3) the premium bond trap and how to avoid it, (4) how the CDS-bond basis links cash and derivative credit markets, and (5) how desks attribute P&L and manage funding costs.

---

## 27.1 Swap Spreads: Definition, Drivers, and Benchmark Dependence

### 27.1.1 The Basic Definition

The swap spread at maturity $T$ is the difference between the par swap rate and the government bond yield at that maturity:

$$\boxed{\text{SS}_T = S_T - y_T^{gov}}$$

This definition appears straightforward, but one practical issue is that a \(T\)-year swap matures in exactly \(T\) years while, at most times, there is no government bond with exactly \(T\) years to maturity. By market convention, the “\(T\)-year swap spread” is quoted versus the \(T\)-year on-the-run government bond. This convention creates the first layer of complexity in interpreting swap spreads.

The maturity mismatch is not merely a technical detail. The on-the-run 10-year Treasury might have 9.8 years remaining; the swap has exactly 10 years. This small difference affects comparisons, particularly when the yield curve is steep.

### 27.1.2 Why Benchmark Choice Matters

The choice of government benchmark has a material impact on the quoted spread. On-the-run Treasury securities can trade at premium prices (lower yields) due to liquidity advantages and special repo financing. As a result, swap spreads quoted off on-the-run benchmarks can be misleading at the few-bp level.

Consider a concrete example. Suppose the 5-year par swap rate is 3.20% and two potential benchmarks exist:

| Benchmark | Yield | Swap Spread |
|-----------|-------|-------------|
| On-the-run 5Y Treasury | 2.95% | 25 bp |
| Fitted off-the-run curve | 3.05% | 15 bp |

The same swap rate produces swap spreads differing by 10 bp purely from benchmark choice. This is not a small discrepancy for relative value trading—10 bp might exceed the entire edge a trader is seeking to capture.

To address this issue, some desks compute swap spreads relative to fitted yields from an off-the-run government curve or explicitly adjust on-the-run yields for financing and liquidity effects. These adjustments require subjective judgement, so “adjusted swap spreads” tend to be desk-specific rather than universally quoted.

> **Practical note:** When analyzing swap spread time series, be aware that the benchmark may change when a new on-the-run issue appears. This can create artificial jumps in the series that have nothing to do with credit or supply/demand dynamics.

### 27.1.3 Why Swap Spreads Are Not Pure Credit Spreads

Swap spreads are sometimes interpreted as “bank credit minus government credit.” This has some truth, but it is incomplete.

The par swap rate is the fixed rate that makes an interest rate swap have zero PV. In standard intuition, it represents what a bank can expect to earn from a series of short-term loans at the floating index (e.g., LIBOR) over the life of the swap—a “continually refreshed” rate. That is not the same object as the yield on a specific term unsecured bank bond.

Furthermore, swap spreads are contaminated by Treasury-side effects that have little to do with bank credit, including:
- on-the-run liquidity and repo specialness (benchmark yields can be rich),
- supply/positioning/financing conditions that cheapen or richen the benchmark, and
- stress/risk-off episodes that move the benchmark yield independently of the swap curve.

The practical implication is that a widening swap spread can come from swap rates rising, Treasury yields falling, or the benchmark issue moving rich/cheap. Attributing the move requires separating the legs and stating the benchmark.

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

## 27.2 Why Swap Spreads Can Be Negative

Swap spreads are simply the difference between swap rates and government bond yields of a given maturity. One practical complication is that a \(T\)-maturity swap matures at exactly \(T\) years while, at most times, there is no government bond with *exactly* \(T\) years to maturity. By convention, therefore, the \(T\)-maturity swap spread is quoted versus the on-the-run government bond at that maturity.

Using the conventions from Section 27.1,

$$\text{SS}_T = S_T - y_T^{gov}$$

where \(S_T\) is the par swap rate to maturity \(T\) and \(y_T^{gov}\) is the yield of the benchmark government bond (often the on-the-run issue closest to \(T\)). With this definition, a negative swap spread simply means \(S_T < y_T^{gov}\).

The “credit paradox” comes from silently replacing \(y_T^{gov}\) with an abstract, frictionless “risk-free curve.” But the benchmark yield is the yield on a particular bond, and that bond can be rich or cheap relative to a fitted government curve. A useful decomposition is:

$$\text{SS}_T^{OTR} = (S_T - y_T^{fit}) - (y_T^{OTR} - y_T^{fit})$$

so the quoted spread depends on both (i) the swap curve versus a smooth government curve and (ii) the benchmark issue’s idiosyncratic rich/cheap.

### 27.2.1 Mechanism 1: The Benchmark Treasury Can Cheapen (Supply/Financing)

By convention, the “10-year swap spread” is the 10-year par swap rate minus the yield of the 10-year on-the-run government bond. Because the benchmark is a specific issue, its yield can move for reasons that do not look like a parallel shift in a smooth government curve.

Two practical cases:
- **On-the-run liquidity and repo specialness can richen the benchmark.** A rich benchmark has a *lower* yield, which mechanically makes \(\text{SS}_T\) look wider.
- **Supply/positioning/financing can cheapen the benchmark.** A cheap benchmark has a *higher* yield, which mechanically tightens \(\text{SS}_T\) and can push it negative.

> **Desk Reality:** The “Treasury” in a swap spread is a *bond*.
> **Common break:** You attribute a swap-spread move to “credit,” but the move came from the benchmark issue richening/cheapening (OTR vs fitted curve, repo specialness, auction supply).
> **What to check:** Compare the benchmark yield to a fitted/off-the-run government curve and look at repo specialness for that issue.

### 27.2.2 Mechanism 2: The Swap Rate Is a “Continually Refreshed” Funding Benchmark

Even if you want to interpret “swap minus Treasury” as a form of credit spread, the par swap rate is not the yield on a term unsecured bank bond. In the standard interpretation, the swap rate represents what a bank can expect to earn from a series of short-term loans to AA-rated borrowers at LIBOR; it is sometimes referred to as a **continually refreshed rate**.

This matters because the “credit content” in the swap curve is tied to the floating index and is effectively *rolling* rather than a static term spread to one issuer. The swap spread’s sign is therefore not a clean statement about term bank credit versus the government.

### 27.2.3 Mechanism 3: Hedging Flows Can Move Swaps Relative to Treasuries

Swap spreads can also move because the swap rate itself moves relative to Treasuries due to hedging flow. One documented “directionality” mechanism is mortgage hedging. For example, if rates fall (say 25 bp) and MBS durations shorten, hedgers may receive fixed in swaps (often at intermediate maturities) to add duration. That receiving pressure can push swap rates down relative to Treasuries and *narrow* swap spreads. When rates rise, the story works in reverse: hedgers may pay fixed, swap rates rise relative to Treasuries, and swap spreads *widen*.

### 27.2.4 Practical Diagnosis: Separate the Legs

When you see a swap spread level or move (especially a negative level), make the diagnosis explicit:
1. **Define the benchmark:** OTR issue vs a fitted/off-the-run curve.
2. **Separate drivers:** did \(S_T\) move, did \(y_T^{gov}\) move, or did the benchmark rich/cheap change?
3. **State curve conventions:** single-curve vs multi-curve assumptions (projection vs discounting) can change the “swap rate” you compare to.

---

## 27.3 Asset Swaps: Structure and Pricing

### 27.3.1 Motivation: Anchoring Bonds to the Swap Curve

For short maturities, traders sometimes compare a bond’s yield to money-market benchmarks. For longer maturities, a common relative-value measure is the **asset swap spread (ASW)**, which anchors a bond to the swap curve rather than to Treasuries.

There are two closely related ways to think about ASW:
1. **As a spread measure:** the spread \(A\) such that discounting the bond cashflows on the swap (LIBOR) curve plus \(A\) reproduces the bond’s price.
2. **As a trading package:** a bond combined with an interest rate swap that converts fixed coupons into floating payments plus a spread.

Because ASW is benchmarked to the swap curve, it avoids subtracting an on-the-run Treasury yield and therefore avoids some on-the-run rich/cheap contamination that affects Treasury-based spreads.

> **Concept: The Synthetic Floating Rate Note**
>
> Think of an asset swap as a way to turn a fixed-rate corporate bond into a synthetic floating-rate note.
>
> - **Problem:** You want exposure to an issuer’s credit, but you prefer floating-rate cashflows.
> - **Trade:** Buy the bond and enter a swap that pays fixed (matching the bond coupon schedule) and receives floating plus a spread.
> - **Result:** The fixed bond coupons and fixed swap payments largely offset, leaving floating + spread (before funding).

### 27.3.2 Asset Swap Mechanics: The Par Asset Swap

In a par asset swap, an investor combines a bond purchase with an interest rate swap structured so that the combined package costs par.

**On settlement (time 0).** The bond is delivered and par is paid. From the asset swap buyer's perspective, the net upfront value is $P - 1$, where $P$ is the bond's full (dirty) price per 1 of face value. Equivalently:
- If $P < 1$ (discount bond), the buyer is paying above the bond’s market value at inception and must be compensated by the running spread.
- If $P > 1$ (premium bond), the buyer receives value upfront and the running spread can be lower (or even negative).

**Interest rate swap.** The buyer enters a payer swap with fixed leg payments matching the bond coupon schedule. On the floating leg, the buyer receives LIBOR plus the asset swap spread $A$.

**Cash flow netting.** The bond coupons received offset the fixed swap payments, leaving the buyer with a net position of receiving LIBOR + $A$ minus the cost of financing the bond in repo.

If the bond defaults, the bond coupon payments cease but the fixed payments on the swap are still due. This asymmetry—bond coupons cease on default but swap payments continue—is a key risk feature of asset swaps that distinguishes them from simple floating rate notes.

The implications are significant. If the bond defaults immediately after the asset swap begins, the investor loses both the bond (now worth only recovery $R$) and continues to owe fixed payments on the swap. This creates exposure beyond the simple credit risk of the bond itself.

### 27.3.3 The Par Asset Swap Spread Formula

Define the Libor-discounted value of the bond's fixed cashflows:

$$P_{\text{Libor}}(0,T) = \frac{c}{f} \sum_{n=1}^{N} Z(0,t_n) + Z(0,T)$$

This is the present value of the bond's fixed cashflows (coupons plus principal) discounted on the Libor/swap curve. Also define the PV01:

$$\text{PV01}(0,T) = \sum_{n=1}^{N} Z(0,t_n) \, \Delta(t_{n-1}, t_n)$$

which represents the present value of receiving 1 unit of spread per year on the floating leg. Note that $Z$ is dimensionless and $\Delta$ is in years, so PV01 has units of "discounted years."

The swap value at initiation is:

$$V(0) = 1 + A(0) \cdot \text{PV01}(0,T) - P_{\text{Libor}}(0,T)$$

The par asset swap structure requires $V(0) + P = 1$, which after rearrangement yields:

$$\boxed{A(0) = \frac{P_{\text{Libor}}(0,T) - P}{\text{PV01}(0,T)}}$$

**Interpretation.** An asset swap can be viewed as long a defaultable bond and short a risk-free bond with the same coupon schedule; the running spread amortizes the price difference over the life of the trade.

**Sign interpretation.** The formula provides clear intuition:
- If $P < P_{\text{Libor}}$ (bond trades cheap relative to the swap curve), then $A > 0$. The buyer receives a positive spread over LIBOR as compensation for taking the credit risk.
- If $P > P_{\text{Libor}}$ (bond trades rich relative to the swap curve), then $A < 0$. The buyer actually pays a spread below LIBOR.

### 27.3.4 Worked Example: Computing the Par Asset Swap Spread

**Example Title**: Par ASW from a bond price and a swap curve

**Context**
- You want a bond-vs-swap-curve spread measure that maps to an actual asset-swap package (bond + swap).
- You want the *units* and *P&L meaning* of “a 1bp move in ASW”.

**Timeline (Make Dates Concrete)**
- Trade date: 2026-02-15
- Settlement date: 2026-02-17 (assume T+2 for illustration)
- Coupon schedule: semiannual on Aug 15 and Feb 15
- Maturity/payment date: 2031-02-15

**Inputs**
- Bond: 5Y maturity, 7.25% annual coupon, semiannual payments (\(f=2\)); face scaled to \$1 (convert back to “per 100” at the end).
- Full (dirty) price: \(94.38\) per \(100\) face, i.e. \(P=0.9438\) per \$1 face.
- Swap/LIBOR-curve analytics (computed from discount factors using the definitions in Section 27.3.3):
  - \(P_{\text{Libor}}(0,T)=1.0876\) per \$1 face.
  - \(\text{PV01}(0,T)=4.4396\) (units: discounted years).

**Outputs (What You Produce)**
- Par asset swap spread \(A(0)\) (bp/year).
- Spread PV sensitivity magnitude: \(\text{PV01}\times 1\text{bp}\) per unit notional.

**Step-by-step**
1. Normalize price to per \$1 face: \(P=94.38/100=0.9438\).
2. Compute \(P_{\text{Libor}}(0,T)\) (PV of fixed bond cashflows discounted on the swap curve) and \(\text{PV01}(0,T)\) (the floating-leg annuity, i.e., discounted accrual factors). For illustration, we use the inputs below.
3. Apply the par ASW formula:
   $$A(0)=\\frac{P_{\\text{Libor}}(0,T)-P}{\\text{PV01}(0,T)}.$$
4. Convert to basis points by multiplying the decimal by \(10{,}000\).

| Parameter | Value |
|-----------|-------|
| Coupon | 7.25% annual, semiannual payments |
| Full price | $94.38 per $100 face |

Suppose curve analytics give $P_{\text{Libor}} = 1.0876$ per dollar of face value, and $\text{PV01} = 4.4396$ discounted years.

Applying the formula:

$$A(0) = \frac{1.0876 - 0.9438}{4.4396} = \frac{0.1438}{4.4396} = 0.0324 = 324 \text{ bp}$$

**Cashflows (per 100 face, ignoring default)**

| Date | Cashflow | Explanation |
|---|---|---|
| 2026-08-15 | +3.625 | coupon \(=100\\times 0.0725/2\) |
| … | +3.625 | every 6 months |
| 2031-02-15 | +103.625 | final coupon + principal |

**P&L / Risk Interpretation**
- The number “ASW = 324 bp” means: in the par asset-swap package, you are effectively receiving floating \(+\) 324 bp (before funding), so the bond is *cheap* versus the swap curve.
- Sensitivity magnitude: \(1\) bp \(=10^{-4}\). Per \$1 notional, a 1bp change in the contractual spread changes PV by about \(\text{PV01}\\times 10^{-4}\). With \(\text{PV01}=4.4396\), that is \(4.4396\\times 10^{-4}=0.00044396\) per \$1 notional (and \(\$4{,}440\) per \(\$10\)mm).
- For an *existing* asset swap, the relevant move is usually the *market* ASW \(A(t)\). Using \(\text{MTM}(t)=(A(0)-A(t))\\cdot\\text{PV01}(t,T)\), a widening in market ASW (higher \(A(t)\)) is a mark-to-market loss for the original buyer.

**Sanity Checks**
- Units check: \(\text{PV01}\) has units of discounted years; multiplying by a spread in 1/year gives a dimensionless PV per \$1 notional.
- Sign check: widening ASW corresponds to bond cheapening (higher required spread); an existing asset-swap buyer loses when ASW widens.
- Repricing check: verify \(V(0)+P\\approx 1\) (below).

**Repricing check.** We can verify: $V(0) = 1 + 0.0324 \times 4.4396 - 1.0876 = 1 + 0.1439 - 1.0876 = 0.0563$. Then $V(0) + P = 0.0563 + 0.9438 = 1.0001 \approx 1$ (the small error is rounding). ✓

### 27.3.5 The Market Asset Swap: An Alternative Structure

The par asset swap creates counterparty exposure when the bond trades away from par. The buyer exchanges par for a bond worth $P$, so the package embeds an exposure to the counterparty when $P$ is far from 1.

Consider a bond trading at 95. The asset swap buyer pays 100 but receives a bond worth only 95. If the asset swap seller defaults immediately, the buyer has lost 5 points.

The market asset swap addresses this by setting the notional equal to the full bond price $P$ rather than par. The market asset swap spread $A^*$ relates to the par asset swap spread by:

$$\boxed{A^*(0) = \frac{A(0)}{P}}$$

For discount bonds ($P < 1$), the market asset swap spread is higher than the par spread because the floating leg notional is smaller. For premium bonds ($P > 1$), it is lower.

The mechanics are:
1. On settlement, the bond price $P$ is paid (not par)
2. The floating leg notional equals $P$
3. At maturity, the floating leg pays $P$ while the fixed leg pays 1, creating a net payment of $(P-1)$

A key feature of the market structure is that it shifts the counterparty exposure away from initiation: instead of exchanging par at inception, the economics are reflected via the terminal $(P-1)$ payment at maturity.

### 27.3.6 Mark-to-Market of an Asset Swap

The mark-to-market value of a par asset swap at time $t$ is:

$$\text{MTM}(t) = (A(0) - A(t)) \cdot \text{PV01}(t,T)$$

where $A(t)$ is the current asset swap spread for the bond. This formula follows from the fact that unwinding an asset swap involves entering an offsetting position at the current spread. The fixed legs and Libor payments cancel, leaving only the difference in spreads times the remaining PV01.

**Worked example.** If the asset swap was entered at $A(0) = 323.9$ bp on $10 million notional, and one year later the spread has tightened to $A(t) = 284$ bp with remaining PV01 of 3.622, the mark-to-market is:

$$\text{MTM} = (323.9 - 284.0) \text{ bp} \times 3.622 \times \$10\text{mm} = 39.9 \times 0.0001 \times 3.622 \times 10{,}000{,}000 = \$144{,}518$$

The position has profited because the issuer’s credit spread tightened, as reflected in both a higher bond price and a lower market ASW.

### 27.3.7 Financing Intuition: Why Repo Matters

In practice, asset swaps are typically financed positions. If you buy the bond, finance it in repo, and enter an asset swap that pays the bond coupon fixed and receives floating + \(A\), then (ignoring default and small timing differences) the fixed bond coupons largely offset the fixed swap payments. The position is left earning approximately:

$$\text{Net} \\approx (\\text{floating index} + A) - r_{\\text{repo}},$$

so the realized carry depends on the repo rate you pay (and whether the bond is “special”).

---

## 27.4 The Premium Bond Trap

### 27.4.1 The Problem: Par ASW Mixes Credit and Price Effects

The par asset swap spread has a subtle but important flaw: it can be misleading for bonds trading far from par. This "premium bond trap" catches practitioners who mechanically compare par ASW spreads across bonds with very different dollar prices and coupons without thinking through the embedded price/par mechanics.

The issue arises from the structure of the par asset swap. When a bond trades at a premium ($P > 100$), the asset swap buyer:
1. Pays par (100) but receives a bond worth $P > 100$
2. Has net upfront value of $(P - 100)$ at inception (because the par exchange differs from the bond’s full price)

This upfront value is effectively amortized into the running spread over the life of the swap. The running ASW quote can therefore reflect not only credit compensation but also the mechanical effect of being far from par.

### 27.4.2 Intuition: Why Par ASW Can Mislead Away from Par

Consider two bonds from the same issuer with identical credit risk:

| Bond | Coupon | Price | Maturity | True Credit Spread |
|------|--------|-------|----------|-------------------|
| Bond A | 2% | 95 | 5Y | 100 bp |
| Bond B | 6% | 108 | 5Y | 100 bp |

Both bonds have the same credit spread in a Z-spread or CDS sense. But their par ASW quotes can differ materially because the par-asset-swap structure embeds an upfront value linked to price and coupon schedule.

- **Bond A (discount):** The structure includes a positive upfront paid by the buyer (about 5 points) that is effectively amortized through the running spread.
- **Bond B (premium):** The structure includes a positive upfront value to the buyer (about 8 points) that is also effectively amortized through the running spread.

As a result, cross-bond comparisons of par ASW can confuse "credit compensation" with "price/par mechanics."

> **Pitfall — Par ASW is a “pure credit spread”:** Par asset swap spreads embed both credit compensation *and* a mechanical price/par amortization effect when bonds are far from par.
> **Why it matters:** You can buy the “wide ASW” bond and discover you mainly bought premium/discount mechanics rather than relative credit value.
> **Quick check:** If \(|P-100|\) is large, compare market ASW \(A^*=A/P\) and a curve-based spread measure (ZVS/Z-spread). If the ranking flips, you are in premium-bond-trap territory.

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

Market ASW ($A^* = A/P$) reduces the mechanical par-notional effect, but ASW remains convention-dependent. For cross-bond comparisons, desks typically cross-check with Z-spread and/or CDS.

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

The zero-volatility spread (ZVS), also called the Z-spread, takes a different approach from asset swap spreads. One definition is: the fixed spread adjustment to the Libor discount rate that reprices the bond.

In discrete compounding form:

$$P = \frac{c}{f} \sum_{n=1}^{N} \frac{1}{(1 + (r(0,t_n) + \theta)/f)^n} + \frac{1}{(1 + (r(0,t_N) + \theta)/f)^N}$$

Or in continuous compounding form:

$$P = \frac{c}{f} \sum_{n=1}^{N} Z(0,t_n) e^{-\theta t_n} + Z(0,T) e^{-\theta T}$$

where $\theta$ is the ZVS to be solved for numerically.

For option-free bonds, ZVS is the natural curve-based spread measure (as opposed to OAS, which is typically used when cashflows are option-dependent). Practitioners often prefer ZVS to yield-based measures because it respects the term structure of discount rates.

### 27.5.2 Conceptual Difference: ZVS vs. ASW

Although ASW and ZVS both measure a bond's spread to the swap curve, they are conceptually different:

| Aspect | ZVS | ASW |
|--------|-----|-----|
| **Mechanics** | Discounting spread: add $\theta$ to all discount rates | Swap-structure spread: spread on floating leg in a package |
| **What it values** | PV of bond cashflows with shifted discount curve | Floating leg receipts in swap-plus-bond package |
| **Solved from** | $P = \sum \text{DF}_{\text{shifted}} \times \text{CF}$ | $V(0) + P = 1$ in par asset swap structure |
| **Numerical method** | Root-finding (iterate $\theta$) | Direct formula |
| **Price sensitivity** | None—defined as the spread that reprices | Par ASW varies with price; market ASW adjusts |

For simple fixed-rate bonds under single-curve assumptions with a flat curve, the two measures are close. In the special case of a flat swap curve, ASW reduces to (bond yield − swap rate); away from that case, ASW and ZVS can differ meaningfully.

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

A common desk definition is “the excess of the CDS spread over the asset swap spread.” This basis connects two markets that should theoretically price the same credit risk—the CDS market and the cash bond market.

In a frictionless world, the basis should be zero: the cost of insuring against default via CDS should equal the credit spread earned by holding the bond. In practice, the basis fluctuates, sometimes significantly, creating relative value opportunities and revealing market stress.

### 27.6.2 Why the Basis Can Move (and Stay Away From Zero)

In a frictionless world, the basis would be near zero because “bond + CDS protection” replicates a near-risk-free position. In practice, this replication is fragile: it requires funding, balance sheet, and bond liquidity.

Large negative bases are most often associated with stressed funding/liquidity conditions where bonds cheapen and the basis trade is hard to execute at scale (e.g., during the 2007–2009 credit crisis).

### 27.6.3 Factors Affecting the Basis

Six common factors that drive the CDS-bond basis are:

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

Asset swap positions carry interest rate risk because the bond’s rates sensitivity typically differs from the swap package’s rates sensitivity. Even if you think of an asset swap as a “credit trade,” day-to-day P&L can still be dominated by rates unless you hedge it.

**Risk definition (explicit convention).** Aligning with `refactor_plan/notation_registry.md`, we use:

$$DV01 := PV(\\text{rates down }1\\text{bp})-PV(\\text{base}),\\qquad 1\\text{bp}=10^{-4}.$$

**Bump object (what is being bumped).** In this chapter’s single-curve examples, “rates down 1bp” means a parallel \(-1\) bp shift to the discounting curve used to value the position (rebuild discount factors consistently). In multi-curve settings you must state which curve(s) are bumped.

**Units and sign.** DV01 is reported in currency per 1bp per stated notional. Positive DV01 means the position benefits when rates fall (typical for a long fixed-rate bond or a receive-fixed swap). A pay-fixed swap has negative DV01 under this convention.

To construct a DV01-neutral hedge, choose the swap notional so net DV01 is (approximately) zero. In practice you usually quote DV01s as positive *magnitudes* and then choose the swap direction (pay vs receive fixed) to offset the bond:

$$\boxed{N_{\\text{swap}} = N_{\\text{bond}}\\times \\frac{DV01_{\\text{bond}}}{|DV01_{\\text{swap}}|}}$$

where \(|DV01_{\\text{swap}}|\) is the *magnitude* of the swap DV01 per unit swap notional under the same bump design.

**Example (with signs).** Consider a 3-year bond with coupon 5%, yield 5.20%, giving \(DV01_{\text{bond}}=+0.0263\) per 100 face under the “rates down 1bp” bump. Suppose a pay-fixed swap of the same maturity has \(DV01_{\text{swap}}=-0.0350\) per 100 notional (negative because pay-fixed loses when rates fall). Then the DV01-neutral hedge ratio uses magnitudes:

$$\text{Swap notional fraction} = \frac{0.0263}{0.0350} = 0.751$$

So $100 million in bonds would be DV01-matched with approximately $75.1 million swap notional.

> **Practical warning:** DV01 matching creates a zero first-order exposure to parallel rate shifts, but residual risks remain—curve twists, convexity differences, and of course spread changes.

### 27.7.2 Spread Sensitivity

There are two closely related “spread sensitivities” that get conflated in practice; it helps to state which one you mean.

**(A) Contractual-spread sensitivity (new trade).** If you hold the curve and cashflows fixed and change the *contractual* asset swap spread \(A\) you receive, the PV changes with:

$$\frac{\partial V}{\partial A} = \text{PV01}$$

Here \(\text{PV01}\) is the annuity \(\sum Z\\Delta\) (units: discounted years). A 1bp change is \(\Delta A=10^{-4}\).

**(B) Market-spread sensitivity (mark-to-market of an existing trade).** If you already entered an asset swap at \(A(0)\), O’Kane’s MTM formula is:

$$\\text{MTM}(t)=(A(0)-A(t))\\cdot\\text{PV01}(t,T),$$

so \(\partial\\text{MTM}/\\partial A(t)=-\\text{PV01}(t,T)\): widening market ASW is a loss for the original buyer.

**Units and magnitude.** For a $100 million position with \(\text{PV01}=2.763\) discounted years, a 1 bp change corresponds to approximately:

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
> A financed asset swap earns roughly \((\text{floating index} + A) - r_{\text{repo}}\) (before default and other frictions). Repo is therefore a first-order driver of realized carry even if the credit spread is unchanged.
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

Even positions framed as “collect the spread” can experience significant mark-to-market volatility from spread changes, potentially forcing position reductions at the worst time.

### 27.7.5 Repo Effects in ASW Trades

Repo rates directly affect ASW carry and can vary for several reasons:

| Repo Effect | Impact on ASW P&L |
|-------------|-------------------|
| **General collateral (GC) rate moves** | Higher GC = lower carry |
| **Bond goes special** | Lower repo = higher carry (bond cheaper to finance) |
| **Fails and scarcity** | Can make repo unpredictable |
| **Quarter-end balance sheet constraints** | Repo rates spike; carry compresses |
| **Credit events affecting collateral eligibility** | May lose repo access entirely |

> **Practitioner Note:** Repo market dynamics are covered in Chapter 9. For ASW positions, the key insight is that “earning floating + spread” is net of repo—and repo can move significantly, especially during stress.

---

## 27.8 Swap-Curve Relative Value Framework

### 27.8.1 The RV Trader's Perspective

Swap-curve relative value is not a single formula but a *practice*—the comparison of bond value to the swap curve rather than to Treasuries. Asset-swap-style measures are used precisely because they benchmark to a curve that is less affected by individual Treasury rich/cheap.

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

Intuition: ASW is meant to measure the bond’s value relative to the swap curve. If the bond is priced close to its swap-curve PV (\(P \\approx P_{\\text{Libor}}\)), the par ASW is near zero. If the bond is cheap versus the swap curve (\(P < P_{\\text{Libor}}\)), ASW is positive; if the bond is rich (\(P > P_{\\text{Libor}}\)), ASW can be low or even negative.

Negative ASW does not require any “issuer better than banks” story. Mechanically, it can arise whenever a bond is rich versus the swap curve and/or when financing is very favorable relative to the floating index.

### 27.8.3 Spread-of-Spreads Trade (OTR vs Off-the-Run)

A common Treasury RV question is: is the on-the-run issue too rich versus a nearby off-the-run issue, once you control for curve exposure? One way to express this is a “spread of spreads”: compare each bond’s yield spread to the swap curve, then take the difference.

- The OTR 5-year was 91.3 bp rich to swaps
- The old 5-year was 75.4 bp rich to swaps
- The spread of spreads was $91.3 - 75.4 = 15.9$ bp

A desk can express this view by putting on two DV01-matched packages (each bond financed in repo plus a swap hedge). If each leg is DV01-matched, the combined position’s first-order rates exposure is small; P&L is driven primarily by changes in the spread of spreads (relative rich/cheap) rather than parallel moves in rates.

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

A fully specified multi-curve ASW number depends on the exact curve-building inputs and CSA terms used by the pricing system. Without those conventions, this chapter focuses on the single-curve formulas and qualitative sensitivities.

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

The same swap rate produces swap spreads differing by 10 bp purely from benchmark choice.

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

The net cashflow is approximately \((\text{LIBOR} + \text{ASW} - r_{\text{repo}})\) (ignoring default and small timing differences).

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
- Government benchmark specified (on-the-run vs. fitted/off-the-run)
- Swap rate confirmed as par swap rate
- Swap curve source identified

**Asset Swap Spread:**
- Par/par vs. market/market convention stated
- Bond price input specified (clean $P_c$ vs. full $P$)
- Settlement and upfront payment handling for off-par bonds
- Discounting curve specified (legacy Libor vs. OIS)

**CDS-Bond Basis:**
- Bond spread measure specified (ASW vs. Z-spread)
- CDS contract terms noted (restructuring clause, currency)
- Funding assumption stated

### Common Pitfalls

| Pitfall | Description |
|---------|-------------|
| Mixing swap spread with ASW | Swap spread is (swap − gov); ASW is (bond vs. swap curve)—not identical |
| ZVS vs. ASW confusion | Different constructions (discounting spread vs. swap-package spread) |
| Clean/dirty mismatch | Using clean price where full price is required (or vice versa) |
| Ignoring default terms | Bond default does not extinguish swap fixed payments |
| Premium bond trap | Par ASW can be misleading far from par; price/coupon mechanics contaminate comparisons |
| Forgetting basis and funding | Treasury specialness affects swap spreads; repo levels affect realized carry |
| Compounding mismatches | Semiannual vs. money-market conventions across curves |
| Assuming negative spreads are "wrong" | Negative swap spreads can reflect benchmark cheapening and convention/curve differences; diagnose which leg moved before calling it a mispricing |

### Verification Tests

**Repricing check.** For a par asset swap, verify $V(0) + P = 1$.

**Sign check.** From $A = (P_{\text{Libor}} - P)/\text{PV01}$, if bond price $P$ falls (bond cheapens), $A$ should rise (widen).

**Unit check.** DV01 is a PV change per 1bp under a stated bump design. If DV01 is quoted “per 100” notional, scale to dollars by $(\text{Notional}/100)\times \text{DV01}$ (and if quoted per 1 notional, scale by $\text{Notional}\times \text{DV01}$).

**Cross-check spreads.** If par ASW differs dramatically from Z-spread, investigate price/par effects.

---

## Summary

This chapter has developed the framework for understanding swap spreads, asset swaps, and swap-curve relative value:

1. **Swap spreads** ($\text{SS}_T = S_T - y_T^{gov}$) compare swap rates to government yields but are contaminated by on-the-run liquidity and specialness effects.

2. **Swap spreads are not pure credit spreads**—they reflect rolling short-term credit embedded in the swap index, benchmark choice (OTR vs fitted/off-the-run), and Treasury supply/financing dynamics.

3. **Negative swap spreads** are possible when the chosen Treasury benchmark cheapens (higher yield) relative to a smooth government curve and/or when the swap curve is being used as an alternative benchmark; the right diagnostic is “which leg moved?”

4. **Asset swap spreads** anchor bonds to the swap curve, measuring value relative to bank-credit-based funding. The formula $A = (P_{\text{Libor}} - P)/\text{PV01}$ provides the par ASW spread.

5. **The premium bond trap**: Par ASW can be misleading for bonds far from par; use market ASW and cross-check with Z-spread/CDS for comparisons.

6. **Market asset swaps** use notional equal to bond price, with $A^* = A/P$, reversing the timing and direction of counterparty exposure.

7. **ZVS** is a discounting spread that differs structurally from ASW, though both measure bond value relative to swaps.

8. **The CDS-bond basis** (CDS spread minus ASW spread) links cash and derivative credit markets; a negative basis often reflects funding/liquidity frictions and other limits to arbitrage.

9. **P&L attribution** must include rates, spread, carry, rolldown, and funding components. Repo effects can be material.

10. **Default asymmetry** is crucial: bond defaults stop coupons but swap obligations continue, creating asymmetric exposure.

---

## Key Concepts

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| Swap spread | $S_T - y_T^{gov}$ | Broad swaps vs. gov indicator, but benchmark-sensitive |
| Benchmark choice (OTR vs fitted) | Which government yield you subtract | Can move quoted spreads by several bp with no change in the swap curve |
| Repo specialness | A specific bond finances below GC in repo | Changes the economics and interpretation of bond-vs-swap RV trades |
| Negative swap spreads | Swap rate < benchmark Treasury yield | Often reflects benchmark cheapening and convention/curve differences, not “banks safer than government” |
| Asset swap (par) | Bond + swap package that costs par | Turns a fixed bond into a synthetic floater + spread (before funding) |
| Premium bond trap | Par ASW comparisons break away from par | Cross-check with market ASW and ZVS/Z-spread (and sometimes CDS) |
| $P_{\text{Libor}}$ | PV of bond cashflows on swap curve | Reference for cheap/rich determination |
| PV01 (annuity) | $\sum Z \Delta$ (discounted years) | Converts a spread move (in bp/year) into PV/P&L |
| ZVS / Z-spread | Spread that reprices the bond under curve discounting | Curve-based spread measure (numerical solve) distinct from ASW |
| Market ASW | $A^* = A/P$ | Reduces counterparty risk for off-par bonds |
| CDS-bond basis | CDS spread − ASW spread | Links cash and derivative credit markets |
| Rolling credit | Swap reflects continually refreshed short-term credit | Why swap ≠ term bank bond yield |

---

## Notation

| Symbol | Meaning | Units / Convention |
|---|---|---|
| \(S_T\) | Par swap rate to maturity \(T\) | rate (per year); compounding/day count per swap convention |
| \(y_T^{gov}\) | Government benchmark yield used for the swap spread | rate (per year); must state OTR vs fitted/off-the-run |
| \(\text{SS}_T\) | Swap spread \(\text{SS}_T=S_T-y_T^{gov}\) | bp (or decimal); quote must specify benchmark |
| \(P_{\text{clean}}, P_{\text{dirty}}\) | Clean/dirty bond price | price per 100 notional (or per 1); \(P_{\text{dirty}}=P_{\text{clean}}+AI\) |
| \(AI\) | Accrued interest | currency per 100 (or per 1) |
| \(Z(0,t)\) | Discount factor to \(t\) | unitless |
| \(\Delta(t_{i-1},t_i)\) | Accrual year fraction | years; day count must be stated |
| \(P_{\text{Libor}}(0,T)\) | Bond PV on the swap/LIBOR discount curve | price per 1 notional |
| \(\text{PV01}(0,T)\) | Annuity \(\sum Z\Delta\) used in ASW formulas | discounted years |
| \(A\) | Par asset swap spread | decimal per year; quoted in bp/year |
| \(A^*\) | Market asset swap spread \(A^*=A/P\) | decimal per year; quoted in bp/year |
| \(\theta\) | ZVS / Z-spread (static spread) | decimal per year; solved numerically |
| \(DV01\) | Rates sensitivity scalar | **Book convention:** \(DV01:=PV(\text{rates down }1\text{bp})-PV(\text{base})\); currency per 1bp; bump object must be stated |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | Define swap spread at maturity $T$. | $\text{SS}_T = S_T - y_T^{gov}$, usually vs. on-the-run gov yield |
| 2 | Why is on-the-run benchmarking problematic? | On-the-run yields include liquidity premiums and special financing effects |
| 3 | What does "continually refreshed rate" mean for swaps? | Swap rates reflect rolling short-term bank credit, not static term credit |
| 4 | Give two mechanisms that can make swap spreads negative without a “credit paradox.” | (i) the chosen Treasury benchmark cheapens vs a smooth curve; (ii) the swap rate and Treasury yield embed different premia and conventions |
| 5 | What is the first diagnostic question when swap spreads move? | “Which leg moved?” (swap curve vs Treasury benchmark yield) and what benchmark was used |
| 6 | Write the par ASW spread formula. | $A = (P_{\text{Libor}} - P)/\text{PV01}$ |
| 7 | What is the premium bond trap? | Par ASW can be misleading when bonds are far from par |
| 8 | How to avoid the premium bond trap? | Prefer market ASW and cross-check with Z-spread/CDS |
| 9 | Define $P_{\text{Libor}}(0,T)$. | $(c/f) \sum Z + Z(T)$—Libor-discounted value of bond cashflows |
| 10 | How does market ASW spread $A^*$ relate to $A$? | $A^* = A/P$ |
| 11 | What is the CDS-bond basis? | CDS spread minus ASW spread |
| 12 | What does a very negative CDS-bond basis indicate? | Often: funding/liquidity frictions and other limits to arbitrage; not a guaranteed “free-money” signal |
| 13 | What happens to swap obligations if the bond defaults? | Bond coupons cease but fixed swap payments remain due |
| 14 | Define ZVS. | Fixed spread adjustment to Libor discount rate that reprices the bond |
| 15 | What is DV01 in this book? | \(DV01 := PV(\text{rates down }1\text{bp})-PV(\text{base})\) for the stated bump object; units currency per 1bp |
| 16 | What five components make up ASW P&L? | Rates, spread, carry, rolldown, funding |
| 17 | How does repo affect ASW carry? | Higher repo = lower carry; "special" repo = higher carry |
| 18 | Why use ASW for longer maturities? | Measures value relative to curves less contaminated by security-specific effects |
| 19 | What does positive ASW indicate? | Bond is cheap vs. the Libor/swap discount curve |
| 20 | Can ASW spreads be negative? | Yes. If a bond trades rich versus the swap curve (e.g., very high-quality issuer and favorable financing), par ASW can be near zero or negative |

---

## Mini Problem Set

1. (Compute) Swap spread: \(S_7=3.55\\%\), \(y_7^{\\text{OTR}}=3.40\\%\). Compute \(\text{SS}_7\) in bp.
2. (Compute) Using Q1, the fitted/off-the-run government yield is \(3.46\\%\). Recompute \(\text{SS}_7\) and the difference vs OTR.
3. (Compute) ASW input hygiene: clean price \(P_{\\text{clean}}=101.20\), accrued interest \(AI=0.30\) (per 100). What full price do you use in ASW formulas?
4. (Compute) Compute PV01 (annuity): semiannual schedule with \(Z=\\{0.99,0.975,0.96,0.945\\}\) and \(\Delta=0.5\).
5. (Compute) Compute \(P_{\\text{Libor}}\): coupon \(c=4\\%\), \(f=2\), same \(Z\), maturity DF \(Z(T)=0.945\).
6. (Compute) Solve par ASW \(A\): use Q3–Q5 with \(P=1.0150\). Compute \(A\) in bp.
7. (Compute) Market ASW: use Q6 and \(P=1.0150\) to compute \(A^*=A/P\) in bp.
8. (Concept) A high-coupon bond (8%) trades at 115 with par ASW 180 bp. A low-coupon bond (3%) from the same issuer trades at 92 with par ASW 90 bp. Which looks “cheaper” on par ASW, and why is that comparison suspect?
9. (Concept) Name two reasons the CDS-bond basis can deviate from zero (sign doesn’t matter).
10. (Desk) Swap spreads tightened 8 bp today. Give two different narratives consistent with that move and one check you would run for each narrative (swap leg vs Treasury benchmark leg).
11. (Desk) A trader says: “swap spreads at \(-20\) bp are free money—Treasuries yield more than swaps!” Explain why this view is flawed.
12. (Desk) Describe how a repo rate spike (e.g., quarter-end) affects asset-swap carry.

### Solution Sketches (Selected)
1. \(\text{SS}_7=(3.55-3.40)\\% = 0.15\\% = 15\\text{ bp}\).
2. \(\text{SS}_7^{\\text{fit}}=(3.55-3.46)\\% = 9\\text{ bp}\). Difference vs OTR: \(15-9=6\\text{ bp}\).
3. Full/dirty price: \(P_{\\text{dirty}}=101.20+0.30=101.50\\) per 100, i.e. \(P=1.0150\\) per 1.
6. \(A=(1.0224-1.0150)/1.935=0.00382\\approx 38.2\\text{ bp}\). (1bp \(=10^{-4}\).)
9. Examples: bond price away from par; CDS counterparty risk; cheapest-to-deliver option; liquidity/funding frictions that prevent arbitrage in stress.
11. You must specify the benchmark and the package: the “Treasury yield” is bond-specific (OTR vs fitted; special/cheap in repo), and the trade is a repo-financed cash position plus a swap plus a hedge design. A negative quote is not, by itself, an arbitrage.

---

## References

- Tuckman & Serrat, *Fixed Income Securities* (“Swap Spreads”; “Liquidity Premiums of Recent Issues”; “Asset Swap Spreads and Asset Swaps”)
- O’Kane, *Modelling Single-name and Multi-name Credit Derivatives* (“Accrued Interest”; “The Zero Volatility Spread”; “The Asset Swap”)
- Hull, *Options, Futures, and Other Derivatives* (“Credit Default Swaps and Bond Yields”)
- Hull, *Risk Management and Financial Institutions* (“CDS-Bond Basis”; “LIBOR/Swap Rates”)
- Andersen & Piterbarg, *Interest Rate Modeling* (collateral discounting motivation; multi-curve curve-building discussion)
- Neftci, *Principles of Financial Engineering* (“Negative basis trades”)
- *Simulation and Optimization in Finance* (“Spread Risk”)
