# Chapter 28: Basis Trades in Rates — OIS-IBOR Basis, Swap Spread RV, Curve RV

---

## Introduction

In August 1998, Long-Term Capital Management held what appeared to be a well-hedged position: long U.S. Treasuries against short interest rate swaps, designed to capture the swap spread while remaining neutral to the level of interest rates. The position was "basis neutral" to first order—until it wasn't. As Russia defaulted and global markets panicked, the swap spread widened violently. LTCM's positions, leveraged to capture small mispricings, became catastrophically unprofitable. The lesson was not that the trade was wrong in some fundamental sense—the spread eventually narrowed—but that the *residual risks* embedded in the trade overwhelmed the primary thesis.

Every basis trade is, at its core, a bet on the difference between two similar-looking instruments. You are long one rate and short another. The instruments appear related—they might share the same maturity, reference the same credit quality, or track the same economic variable—but they are not identical. The spread between them is the *basis*, and movements in this basis determine your profit or loss.

This sounds simple, but basis trades are among the most treacherous in fixed income. A trader who believes they have isolated a clean relative value opportunity often discovers, painfully, that they are exposed to risks they never intended to take. As Tuckman emphasizes in his analysis of swap spreads: "While quoted swap spreads are useful for investigating broad themes... in the small these data can be misleading." A swap spread trade is not just a bet on "credit spreads." It is simultaneously a bet on Treasury financing, on-the-run dynamics, regulatory capital treatment, mortgage hedging flows, and a dozen other factors that may not be apparent at inception.

This chapter develops a systematic framework for analyzing basis trades in rates:

1. **The "Long/Short" Decomposition** (Section 28.1): How to break any trade into its fundamental exposures before entering
2. **The Swap Spread Trade** (Section 28.2): What it really bets on, and why it's more complex than "credit vs. risk-free"
3. **The OIS-IBOR Basis** (Section 28.3): The discounting vs. projection spread and how the multi-curve framework creates new basis exposures
4. **Treasury Futures Basis** (Section 28.4): A detailed worked example of basis trade mechanics
5. **Curve Relative Value** (Section 28.5): Trading the "spread of spreads" across the term structure
6. **Hidden Risks** (Section 28.6): Funding, roll, liquidity, and the exposures that traders forget

The material builds directly on Chapter 20's multi-curve framework and Chapter 27's asset swap mechanics. This chapter focuses not on the *definition* of these spreads but on *how traders use them*—and what can go wrong.

---

## 28.1 The "What You're Long / What You're Short" Framework

### Every Trade Has a Decomposition

Before entering any basis position, a trader should be able to articulate precisely what they are buying and what they are selling. This sounds obvious, yet the failure to decompose trades properly has destroyed more P&L than almost any other analytical failure in fixed income.

Tuckman emphasizes this principle in the context of Treasury futures basis trades: "the P&L from the long basis position equals the size of the bond position times the change in the net basis." But understanding *what drives* the net basis requires decomposing the trade into its fundamental exposures. The formula tells you how to calculate P&L; decomposition tells you *why* you will make or lose money.

Consider a trader who wants to "buy the swap spread"—that is, bet that swap rates will fall relative to Treasury yields. The naive decomposition is simple: long credit risk, short risk-free rates. But this misses critical details:

- Which Treasury are you shorting? On-the-run or off-the-run?
- How are you financing the short Treasury position?
- What happens when the on-the-run Treasury "rolls off" and a new issue becomes the benchmark?
- Is the swap collateralized? If so, what currency is the collateral?

Each of these questions reveals a hidden exposure that can overwhelm the primary trade thesis.

### A Framework for Decomposition

For any rates basis trade, there are typically three layers of exposure:

| Layer | What It Represents | Examples |
|-------|-------------------|----------|
| **Level Risk** | Directional exposure to the overall level of rates | Unhedged DV01, curve parallel shift |
| **Spread Risk** | Exposure to the spread between two rate curves | Swap spread, OIS-IBOR basis, tenor basis |
| **Residual Risk** | Exposures that remain after hedging | Funding, roll, convexity, liquidity, positioning |

A well-structured basis trade should have *zero* level risk (be DV01-neutral), *intentional* spread risk (the view being expressed), and *minimized* residual risk (or at least, understood and accepted residual risk). The decomposition process forces the trader to verify that each layer is appropriately managed.

### The Decomposition Process

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

Even a "DV01-neutral" hedge is never perfect. Tuckman's analysis of trading TED spreads illustrates this: "Without the DV01 hedge, the net position in Eurodollar futures contracts would not be zero and the trade would make or lose money if bond prices stayed the same while all futures rates rose or fell by one basis point."

The residual exposures include:
- **Curve shape risk**: Steepening/flattening affects bonds and swaps differently
- **Funding risk**: Financing the position costs money; rates can change
- **Roll/carry**: The trade ages; time decay and curve roll matter
- **Convexity differences**: Second-order effects diverge in large rate moves
- **Liquidity/specialness**: Specific instruments may cheapen or richen unexpectedly

**Step 4: Quantify the Residuals**

The critical question: If my primary view is correct (the spread moves as expected), but each residual risk moves against me by a reasonable amount, do I still make money? If the answer is "maybe not," the trade is poorly structured.

---

## 28.2 The Swap Spread Trade: Anatomy and Hidden Complexity

### What Is a Swap Spread?

Tuckman defines swap spreads as "simply the difference between swap rates and government bond yields of a particular maturity." The 10-year swap spread is the 10-year par swap rate minus the 10-year on-the-run Treasury yield.

On the surface, this measures the credit premium of swaps (which historically referenced bank credit via LIBOR, now referencing SOFR) over Treasuries. Tuckman provides historical context: "The fall in swap spreads in the early 1990s reflected the recovery of the banking sector from its problems in the 1980s." But this simple interpretation—swap spreads as a credit barometer—conceals substantial complexity.

### What Actually Drives Swap Spreads?

Swap spreads reflect multiple factors, not just credit risk:

**1. Credit Risk of the Banking Sector**

Historically, LIBOR incorporated bank credit risk; Treasuries are considered "risk-free." Even in the post-LIBOR SOFR world, the swap market reflects dealer credit conditions through funding costs. But credit alone cannot explain swap spread dynamics.

**2. Supply and Demand for Treasuries**

Treasury scarcity pushes yields down, widening swap spreads. Tuckman documents this effect: "The rise in swap spreads in the late 1990s... can be best explained by a perceived scarcity in the supply of U.S. Treasury securities relative to demand." The U.S. budget surpluses of the late 1990s reduced Treasury issuance, making Treasuries more expensive and widening swap spreads.

**3. Repo Financing Dynamics**

On-the-run Treasuries trade "special" in the repo market—their repo rates are below general collateral rates because of high borrowing demand. As Tuckman explains: "Investors and traders long an on-the-run security for liquidity reasons require compensation to sacrifice liquidity by lending those securities in the repo market. At the same time, investors and traders wanting to short the on-the-run securities are willing to pay for the liquidity of shorting these securities when borrowing them in the repo market."

This financing advantage lowers the effective yield of on-the-run Treasuries, inflating measured swap spreads.

**4. Regulatory Capital**

Swaps and Treasuries have different capital treatments under Basel III and other regulatory frameworks. Changes in these rules can shift the relative demand for swaps versus Treasury hedging.

**5. Mortgage Hedging Flows**

GSEs, mortgage servicers, and mortgage REITs use swaps extensively to hedge the duration of mortgage-backed securities. Tuckman documents this effect in detail: "Given the size and convexity of the universe of MBS, and an estimate of how much of that market is actively hedged, a 25 basis point change in the swap rate requires a hedge adjustment of about $30 billion face amount of 10-year swaps."

This creates *directional behavior* in swap spreads: when rates fall, MBS duration shortens and hedgers receive fixed in swaps (narrowing spreads); when rates rise, MBS duration extends and hedgers pay fixed (widening spreads).

### The "Classic" Swap Spread Trade

**Trade**: Receive fixed on a swap, short the corresponding maturity Treasury

**Intended Bet**: Swap spreads will narrow (swap rate will fall relative to Treasury yield)

**Execution**:
- Receive fixed on $100MM 10Y swap at, say, 4.50%
- Short $X face of 10Y Treasury (sized to match DV01)
- Fund the short Treasury position via repo

**What You're Long**:
- Treasury yield rising relative to swap rate
- "Credit spread narrowing" (loosely speaking)

**What You're Short**:
- Swap rate falling relative to Treasury yield
- "Credit spread widening"

### Hidden Exposures in the Swap Spread Trade

Tuckman explicitly warns that "while quoted swap spreads are useful for investigating broad themes... in the small these data can be misleading."

**Exposure 1: On-the-Run / Off-the-Run Dynamics**

The swap spread is typically quoted against the *on-the-run* Treasury. But on-the-run securities have special characteristics that contaminate the spread:

> "Liquidity premiums and special financing have a large impact on the yield of on-the-run government bonds. This means that the swap spread is not a clean measure of the credit of the banking system and of demand and supply in the swap and government bond markets." — Tuckman

If you short the on-the-run 10Y to trade swap spreads, you are also implicitly long the "on-the-run premium." When that premium compresses (as the bond ages off-the-run and a new issue takes its place), you may lose money even if your credit view is correct.

**Exposure 2: Repo Financing**

Shorting the Treasury requires borrowing it in repo. If the bond goes "special" (trades at a repo rate well below general collateral), your financing cost increases. The 5-year on-the-run/off-the-run case study from Tuckman shows the magnitude: the spread between OTR and old five-year term repo rates was 119.2 basis points.

**Exposure 3: DV01 Mismatch Over Time**

Swap DV01 and Treasury DV01 drift differently as rates move and time passes. A trade that starts DV01-neutral will not remain so, requiring periodic rebalancing.

**Exposure 4: Convexity Mismatch**

Treasuries and swaps have different convexity profiles. In large rate moves, these second-order effects can generate significant unexpected P&L.

### Better Ways to Trade Swap Spreads

To avoid contamination by on-the-run dynamics, practitioners often:

1. **Use Off-the-Run Treasuries**: Trade the spread to a seasoned Treasury, accepting less liquidity but cleaner economics
2. **Trade Asset Swaps**: Use the asset swap structure (Chapter 27) to isolate the credit component more precisely
3. **Trade "Adjusted" Swap Spreads**: Explicitly adjust for financing advantages and liquidity premiums, though as Tuckman notes, "Both of these solutions require a good deal of subjective judgement"

---

## 28.3 The OIS-IBOR Basis: Discounting vs. Projection

### What Is the OIS-IBOR Basis?

As developed in Chapter 20, the post-crisis derivatives world uses separate curves for different purposes:
- **OIS Curve**: Used for *discounting* collateralized cashflows
- **IBOR Projection Curves**: Used for *estimating* future floating rate fixings

The spread between these curves is the **OIS-IBOR basis** (or in the modern SOFR-dominated world, the spread between term rates and overnight rates).

Andersen and Piterbarg document the magnitude of this basis during the crisis: "While the spread between the Fed funds rate and 3 month Libor rate used to be very small—in the order of a few basis points—after September 2007 it went up to as much as 275 basis points, and it is now generally accepted that the Libor rate is no longer a good proxy for a discounting rate on collateralized trades."

This basis is not an anomaly to be arbitraged away; it reflects genuine economic differences between overnight and term funding.

### Why the Basis Exists

Chapter 20 covered the economic drivers of tenor basis. For the OIS-IBOR basis specifically:

**Credit Horizon**: A bank borrowing for 3 months faces more credit risk than rolling overnight funding. The term rate must compensate for this additional risk.

**Liquidity Preference**: Banks structurally prefer to match their funding duration to their assets. The desire for term funding creates a premium.

**Counterparty Concentration**: Term lending concentrates credit exposure to a single counterparty; overnight lending allows daily reassessment.

### Trading the OIS-IBOR Basis

**Instruments**: OIS swaps, Fed funds/SOFR basis swaps

**Trade Structure**: Enter a basis swap receiving Term SOFR (or historically LIBOR) and paying OIS + spread

**What You're Long**:
- Bank credit risk / term funding premium
- Liquidity premium in term rates
- Widening of the basis during stress

**What You're Short**:
- The "risk-free" overnight rate
- The "flight to safety" trade

### Risk Decomposition in the Multi-Curve Framework

Andersen and Piterbarg provide a clean decomposition of sensitivities in the multi-curve world:

> "Perturbations to instruments used in building the base index curve... define risk sensitivities to the overall levels of interest rates... Perturbations to funding instruments define sensitivities to discounting. Perturbations to basis swap spreads... define basis risk, i.e. the risk that index curves of different tenors do not move in lock step."

This gives us three orthogonal risk factors:

| Risk Type | What Drives It | Hedge Instrument |
|-----------|---------------|------------------|
| **Rate Level** | Overall rates move up/down | Standard swaps, futures |
| **Discounting** | OIS curve moves | OIS swaps |
| **Basis** | Spread between curves moves | Basis swaps |

The framework allows traders to isolate specific risk exposures while hedging unwanted ones.

### Why OIS-IBOR Basis Moves

**The basis widens when:**
- **Credit stress**: Bank credit concerns push up term rates relative to OIS
- **Liquidity hoarding**: Banks prefer to hold cash rather than lend at term tenors
- **Quarter-end/year-end**: Regulatory constraints tighten bank balance sheets

**The basis narrows when:**
- **Risk-on sentiment**: Credit concerns abate
- **Abundant liquidity**: Central bank operations flood the system with reserves
- **Reduced interbank borrowing**: With ample reserves, banks don't need to borrow at term

### Practical Considerations for Basis Trading

**Correlation with Other Positions**: OIS-IBOR basis is often *correlated* with other credit spreads. A position that is long basis may perform poorly precisely when your other spread positions (corporate bonds, CDS protection sales) also suffer. This is wrong-way risk in disguise.

**Mark-to-Market Volatility**: Basis positions can show significant interim P&L swings. Andersen's 275bp figure illustrates the potential magnitude. Even if your long-term view is correct, position limits or margin calls may force liquidation at the worst time.

---

## 28.4 Treasury Futures Basis: A Detailed Example

Treasury futures provide a concrete, quantifiable example of basis trade mechanics. The structure is rich enough to illustrate the key concepts while being simple enough to analyze precisely.

### Gross Basis and Net Basis

Tuckman defines these precisely. Let $P^i(t)$ be the spot price of bond $i$ at time $t$, let $P^i_{fwd}(t)$ be its forward price to the last delivery date, let $F(t)$ be the futures price, and let $cf^i$ be the conversion factor. Then:

$$\boxed{GB^i(t) = P^i(t) - cf^i \times F(t)}$$

$$\boxed{NB^i(t) = P^i_{fwd}(t) - cf^i \times F(t) = GB^i(t) - \text{carry}^i(t)}$$

Tuckman explains the terminology: "The right-hand side of equation (20.12) explains the term net basis: It is the gross basis net of carry."

At delivery, the forward price equals the spot price (carry equals zero), so gross basis equals net basis. Furthermore, both measures equal the cost of delivery.

### The Long Basis Trade

A long basis trade in bond $i$, as Tuckman describes:

> "Buy $G^i$ face amount of deliverable bond $i$. Sell the repo of bond $i$ to the last delivery date. Sell $cf^i \times G^i / 100{,}000$ futures contracts."

This is equivalent to buying the bond forward and selling the futures. The trade requires no net cash outlay: the repo position finances the bond purchase, and the futures trade requires only margin.

### P&L Formula

The P&L from a long basis trade is remarkably simple:

$$\boxed{P\&L = G^i \times [NB^i(t') - NB^i(t)]}$$

"In words, (20.15) says that the P&L from the long basis position equals the size of the bond position times the change in the net basis."

This formula encapsulates everything: you profit if the net basis widens, lose if it narrows.

### What the Net Basis Represents

The net basis captures the value of the delivery options embedded in the futures contract:

**1. Quality Option**: The short can deliver any eligible bond, choosing the cheapest-to-deliver (CTD)

**2. Timing Option**: The short can choose when during the delivery month to deliver

**3. End-of-Month Option**: After the last trading day, the short can still deliver based on the final settlement price while bond prices may have moved

Tuckman explains: "If the net basis of any bond is near zero, then the quality option embedded in the contract is nearly worthless and selling that bond forward is equivalent to selling the futures contract."

### The "What You're Long/Short" for Futures Basis

**Long Basis Trade**:
- **Long**: The delivery options embedded in the futures contract—the optionality that allows the short to choose which bond to deliver and when
- **Short**: The futures contract's "cheapness" or "richness" relative to fair value

**Short Basis Trade** (the reverse):
- **Short**: The delivery options (you are implicitly short vol)
- **Long**: Any mispricing in the futures contract

### Worked Example: Scenario Analysis

Tuckman provides a detailed case study. Traders sold the 4.75s of November 15, 2008, net basis at 7.45 ticks into TYM0. Table 20.7 in Tuckman shows the scenario analysis:

| Parallel Shift | Futures Price | Nov '08 Net Basis | Basis P&L ($) |
|----------------|---------------|-------------------|---------------|
| -80 bp | 100.37 | 13.2 ticks | -$179,688 |
| 0 bp | 95.49 | 1.0 ticks | +$201,563 |
| +80 bp | 90.52 | 1.1 ticks | +$198,438 |

The trade profits if rates stay unchanged or rise (basis narrows toward zero), but loses if rates rally significantly (basis widens as shorter-duration bonds become CTD).

To hedge the rally risk, traders could purchase futures call options. The case study shows that "the explanation at the time for the cheapening of the futures contract from April 3, 2000, to April 10, 2000, was that many traders were forced to liquidate short basis positions. Since such liquidations entail selling futures and buying bonds, enough activity of this sort will cheapen the contract relative to bonds."

This illustrates the positioning risk inherent in basis trades: when everyone has the same trade, forced liquidation becomes self-reinforcing.

---

## 28.5 Curve Relative Value: The Spread of Spreads

### What Is a Spread of Spreads Trade?

Often the absolute level of a spread is uncertain, but the *relative* relationship between two spreads appears mispriced. This leads to "spread of spreads" trades—positions designed to profit from the convergence of two spreads regardless of their absolute levels.

Tuckman discusses this explicitly in the context of TED spreads: "This trade is typically designed not to express an opinion about the absolute level of TED spreads but, rather, to express the opinion that the TED of the 4.75s of November 15, 2003, is too high relative to that of the 4s of August 15, 2003. In trader jargon, this trade is usually intended to express an opinion about the spread of spreads."

### Example: Trading TED Spread Differentials

**Setup**: Two bonds have different TED spreads (spreads to Eurodollar/SOFR futures rates):
- Bond A: TED spread of 15.6 bp
- Bond B: TED spread of 20.5 bp

**Trade**: Buy Bond A, sell Bond B, hedge each with the appropriate futures strip

**What You're Long**:
- Bond A richening relative to Bond B
- The "spread of spreads" narrowing

**What You're Short**:
- Bond B's cheapness relative to Bond A

**Critical Hedge Construction**: As Tuckman emphasizes, "Without the DV01 hedge, the net position in Eurodollar futures contracts would not be zero and the trade would make or lose money if bond prices stayed the same while all futures rates rose or fell by one basis point."

The trade must be constructed so that:
1. Each bond is hedged against its own rate risk using futures
2. The combined DV01 of both positions is zero
3. The net futures position is approximately zero

Only then does the trade generate P&L solely from the spread of spreads.

### The Five-Year OTR/Old Five-Year Trade

Tuckman's Appendix 18A provides a detailed case study of trading the spread between on-the-run and off-the-run five-year Treasuries via asset swaps.

**Setup on January 17, 2001:**
- OTR 5Y (5.75% Nov 2005): Yield 4.852%, Spread to swaps -91.3 bp
- Old 5Y (6.75% May 2005): Yield 4.971%, Spread to swaps -75.4 bp
- Spread of spreads: -15.9 bp (OTR was 15.9 bp richer to swaps)

**The Question**: Is a spread of spreads of 15.9 basis points justified? Both bonds are Treasuries; the swap curve over six months reflects nearly identical rolling bank credit. The experienced trader argued that on a forward basis to May 8, 2001, accounting for financing and liquidity effects, the spread of spreads should be only about 3 basis points (the typical old-to-double-old premium).

**The Trade**: Sell the OTR asset swap, buy the old 5Y asset swap, sized so DV01s match.

**Result**: "The spread of spreads fell so that the OTR five-year was 2.9 basis points rich to the old five-year. On January 17, 2001, that spread had been sold forward to May 8, 2001, at 7.6 basis points. Therefore, on a position short $100,000,000 5.75s of November 15, 2005, with a forward DV01 of .04086, the profit from the trade was approximately" $192,000.

### Swap Curve Relative Value

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

## 28.6 Hidden Risks in Basis Trades

### Funding Risk

Almost every basis trade requires financing. Changes in funding costs directly affect P&L.

Tuckman illustrates this with asset swap mechanics: "The net cash flow from the asset swap trade... is LIBOR plus 15 basis points minus the repo rate."

If you enter a basis trade earning a spread over financing, and financing rates spike, your periodic income shrinks or turns negative. This is particularly dangerous for leveraged positions where the spread earned is small relative to the total financing cost.

### Roll and Carry

Basis trades age. The passage of time affects P&L through multiple channels:

**Curve roll**: If the yield curve is upward sloping, instruments may "roll down" to lower yields as time passes. A long bond position benefits from this; a swap position may or may not, depending on the curve shape.

**Carry**: Coupon income minus financing cost determines whether a position earns or bleeds money while you wait for your spread thesis to materialize.

**Convergence**: Futures basis must converge to the cost of delivery at expiration. A long basis position that is held to delivery will earn the initial net basis (minus the delivery option value at expiration).

### Liquidity Risk: The Crowded Trade Problem

Tuckman's FNMA case study provides a stark illustration:

> "Many investors and traders had reasoned along the lines of the previous paragraph, had taken account of recent history, and had bought the agency against swaps at asset swap spreads of 0, 10, 15, and so on. As spreads continued to widen, these positions lost a good deal of money and forced liquidations that, in turn, widened the spreads even further."

This is the danger of "crowded trades"—when everyone has the same basis position, liquidation pressure becomes self-reinforcing. The trade may be fundamentally correct (FNMA was extremely unlikely to default), but the path can be devastating. The asset swap spread of the FNMA 6.25s of May 15, 2029 widened from near zero to over 50 basis points before eventually normalizing.

### Convexity Mismatches

For large rate moves, the different convexity profiles of the instruments in a basis trade create P&L that was not part of the original thesis.

The most significant example is mortgage hedging. Tuckman documents the mechanics: when rates fall, MBS duration shortens (negative convexity), and hedgers receive fixed in swaps to compensate. This concentrated receiving pressure narrows swap spreads. When rates rise, the reverse occurs—hedgers pay fixed, widening spreads.

The implication: swap spreads exhibit "directionality." They tend to widen in selloffs and narrow in rallies, not because of credit fundamentals, but because of the convexity hedging needs of the MBS market. A trader who is short swap spreads (receiving fixed, short Treasuries) is implicitly short the volatility of the mortgage market.

### Credit Event Risk

For trades involving credit-risky instruments, the basis can collapse or explode on credit events. O'Kane discusses the CDS-bond basis drivers: "If the agency does default, then the coupon payments from the bond cease but the fixed payments to the swap are still due."

An asset swap holder is exposed to:
- Loss of the bond position in default
- Continued swap payment obligations
- Mark-to-market losses during distress

The CDS-bond basis exists in part to compensate for these asymmetries.

---

## 28.7 Practical Execution Considerations

### Sizing the Trade

Basis trades should be sized based on three factors:

**Maximum drawdown tolerance**: How much interim loss can you sustain without being forced to exit? The FNMA case shows that even fundamentally sound trades can experience drawdowns of 50+ basis points.

**Bid-offer costs**: Basis trades often have multiple legs; each has execution cost. A trade earning 5bp that costs 2bp to enter and 2bp to exit has thin economics.

**Margin and capital**: Futures require margin; swaps require capital under regulatory frameworks. Size constraints often bind before risk limits.

### Entry and Exit

**Entry**: Look for both technical dislocations (forced selling, quarter-end distortions) and fundamental mispricings. The best basis trades combine a sound economic thesis with a catalyst that will cause convergence.

**Exit**: Define the exit criteria before entry:
- Target profit level
- Stop-loss level
- Time horizon (some basis trades "work" only if held to maturity or to delivery)

### Monitoring Residuals

Throughout the life of the trade:
- Rebalance DV01 as needed (hedge ratios drift)
- Monitor funding costs
- Track positioning indicators (is everyone doing this trade?)
- Watch for regime changes that alter the thesis

---

## Summary

1. **Every basis trade has a decomposition**: Before entering, articulate precisely what you are long, what you are short, and what residual risks remain after hedging. Use the three-layer framework: level risk, spread risk, residual risk.

2. **Swap spreads are not "pure credit"**: They reflect Treasury supply/demand, repo specialness, regulatory capital, and mortgage hedging flows. The on-the-run benchmark contaminates the measurement. Tuckman warns that quoted swap spreads can be "misleading in the small."

3. **The OIS-IBOR basis** separates discounting from projection: Trading it means taking a view on bank credit, liquidity conditions, and the term structure of funding. Andersen documents the potential magnitude: 275bp during the crisis.

4. **Treasury futures basis** captures delivery option value: The net basis equals the value of the short's delivery options minus any mispricing. P&L follows the simple formula: position size × change in net basis.

5. **Spread of spreads trades** isolate relative value: They profit from convergence between two spreads, not from directional views on either spread individually. Proper construction requires offsetting DV01 exposures.

6. **Hidden risks abound**: Funding, roll, liquidity, convexity mismatches, and positioning risk can overwhelm the primary thesis. Understanding these residual exposures is as important as understanding the intended bet.

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **Gross Basis** | Spot price − (CF × Futures price) | Starting point for futures basis analysis |
| **Net Basis** | Forward price − (CF × Futures price) = Gross basis − Carry | The actual value being traded in a basis position |
| **Swap Spread** | Par swap rate − Treasury yield | Key measure of rates market dynamics, but not purely credit |
| **OIS-IBOR Basis** | Term rate − OIS rate for equivalent tenor | Captures bank credit/liquidity premium in funding |
| **Spread of Spreads** | Difference between two spread measures | Allows trading relative value without directional views |
| **Long/Short Decomposition** | Breaking a trade into its component exposures | Essential for understanding true risk before entry |
| **Residual Risk** | Exposures remaining after hedging | Often the source of unexpected P&L |

---

## Notation for This Chapter

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

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is the fundamental question before entering any basis trade? | What am I long? What am I short? What residuals remain after hedging? |
| 2 | What is the formula for net basis in Treasury futures? | $NB^i = P^i_{fwd} - cf^i \times F$ (Forward price minus conversion factor times futures price) |
| 3 | Why is the swap spread not a "pure" measure of bank credit? | It also reflects Treasury supply/demand, repo specialness, regulatory capital, and mortgage hedging flows |
| 4 | What three risk types exist in the multi-curve framework according to Andersen? | Rate level risk, discounting risk, and basis risk |
| 5 | What is a "spread of spreads" trade? | A trade that profits from convergence between two spreads, neutral to the absolute level of either |
| 6 | What does a long basis trade in Treasury futures bet on? | That the delivery options embedded in the futures contract have value that will be realized |
| 7 | Why did the OIS-IBOR basis widen dramatically in 2007-2008? | Bank credit concerns and liquidity hoarding drove term rates up relative to overnight rates—to 275bp per Andersen |
| 8 | What is the P&L formula for a Treasury futures basis trade? | P&L = $G^i \times [NB^i(t') - NB^i(t)]$, or position size times change in net basis |
| 9 | Why is using on-the-run Treasuries problematic for swap spread trades? | The on-the-run premium and special repo contaminate the spread; it's not purely about credit |
| 10 | What is "funding risk" in a basis trade? | The risk that financing costs change during the life of the trade, affecting the carry earned |
| 11 | How should a spread of spreads trade be DV01-constructed? | Each leg hedged so the combined DV01 is zero; profits only from spread convergence |
| 12 | What is the "crowded trade" problem illustrated by the FNMA case? | When many traders have the same position, forced liquidation becomes self-reinforcing |
| 13 | How does the passage of time affect a Treasury futures basis trade? | The net basis must converge to the cost of delivery at expiration |
| 14 | What residual risks remain even in a "DV01-neutral" basis trade? | Curve shape, convexity, funding, roll, and liquidity/positioning risks |
| 15 | What market factor creates "directionality" in swap spreads? | Mortgage hedging: hedgers receive fixed when rates fall (narrowing spreads), pay fixed when rates rise (widening spreads) |
| 16 | What does Tuckman say about quoted swap spreads for detailed analysis? | "While quoted swap spreads are useful for investigating broad themes... in the small these data can be misleading" |
| 17 | In the multi-curve world, what do perturbations to basis swap spreads measure? | Basis risk—the risk that index curves of different tenors do not move in lock step |
| 18 | Why might a fundamentally correct basis trade still lose money? | Residual risks (funding, positioning, convexity) can overwhelm the primary thesis during adverse mark-to-market moves |

---

## Mini Problem Set

### Problem 1 (Basic)
A Treasury futures basis trade has initial net basis of 8 ticks. The trade is for $50 million face. If the net basis narrows to 3 ticks, what is the P&L?

*Solution:* P&L = $50MM × (8 - 3)/32 / 100 = $50MM × 5/3200 = $78,125 profit. (A short basis trader profits when the basis narrows; a long basis trader would lose this amount.)

### Problem 2 (Basic)
The 10Y swap rate is 4.50% and the 10Y on-the-run Treasury yields 4.05%. What is the swap spread? If the on-the-run trades 10bp special in repo, how does this affect interpretation?

*Solution:* Swap spread = 4.50% - 4.05% = 45bp. The 10bp repo specialness artificially lowers the Treasury yield (investors accept lower yields for the financing advantage). Adjusting for specialness, the "true" Treasury yield might be closer to 4.15%, giving an adjusted swap spread of about 35bp. Per Tuckman, the quoted spread is "misleading in the small."

### Problem 3 (Intermediate)
You enter a swap spread trade: receive fixed on $100MM 10Y swap, short $95MM 10Y Treasury (sized for DV01 match). The Treasury goes very special, costing you 15bp annualized in repo vs. GC. How does this affect your trade over 3 months?

*Solution:* Additional financing cost = $95MM × 0.15% × (3/12) = $35,625. This is a hidden drag on P&L regardless of what happens to swap spreads. If your spread thesis earns less than this, the trade loses money even if you're "right."

### Problem 4 (Intermediate)
Bond A has a TED spread of 20bp with DV01 of $4,500 per $1MM. Bond B has a TED spread of 30bp with DV01 of $5,000 per $1MM. You want to trade the spread of spreads with $10MM of Bond B. How much of Bond A do you need?

*Solution:* Bond B total DV01 = $10MM × $5,000/$1MM = $50,000. To match: need Bond A DV01 = $50,000. So face = $50,000 / $4,500 × $1MM = $11.11MM of Bond A. You buy $11.11MM of Bond A (cheap to TED) and sell $10MM of Bond B (rich to TED).

### Problem 5 (Advanced)
Explain why a DV01-neutral swap spread trade is still exposed to curve steepening/flattening risk. How would you hedge this residual exposure?

*Solution:* The swap and Treasury have different DV01 profiles across the curve (different key rate durations). A swap has distributed coupon exposure; a Treasury has concentrated maturity exposure. A steepening move might benefit the Treasury more than the swap, or vice versa, creating P&L even at zero aggregate DV01. To hedge: decompose into key rate DV01 buckets and hedge each, or use additional curve instruments (e.g., a 5s10s steepener/flattener overlay).

### Problem 6 (Advanced)
In the multi-curve framework, a portfolio has:
- $10MM pay-fixed 5Y swap referencing 3M Term SOFR
- $10MM receive-fixed 5Y OIS swap

What are the three risk sensitivities (rate level, discounting, basis) of this combined position?

*Solution:*
- **Rate level**: Both swaps have similar DV01 magnitude but opposite signs → approximately zero rate level risk
- **Discounting**: Both are discounted at OIS → remaining sensitivity to OIS curve moves, but largely offset
- **Basis**: The pay-fixed Term SOFR swap is sensitive to the Term SOFR-OIS basis; the receive-fixed OIS swap is not directly exposed. Net position: long Term SOFR-OIS basis (you profit if Term SOFR rises relative to OIS).

---

## Source Map

### (A) Verified Facts (Source-Backed)

| Fact | Source |
|------|--------|
| Net basis formula: $NB^i = P^i_{fwd} - cf^i \times F$ | Tuckman Ch 20, eq. 20.11 |
| Net basis P&L formula: $G^i \times [NB^i(t') - NB^i(t)]$ | Tuckman Ch 20, eq. 20.15 |
| "The P&L from the long basis position equals the size of the bond position times the change in the net basis" | Tuckman Ch 20 |
| Swap spread = swap rate − Treasury yield definition | Tuckman Ch 18 |
| "Swap spreads are closely watched indicators of fixed income market pricing" | Tuckman Ch 18 |
| Rise in swap spreads late 1990s due to Treasury scarcity | Tuckman Ch 18 |
| "While quoted swap spreads are useful for investigating broad themes... in the small these data can be misleading" | Tuckman Ch 18 |
| On-the-run yields contaminated by liquidity premiums and special financing | Tuckman Ch 18 |
| OIS-IBOR spread widened to 275bp in September 2007 crisis | Andersen Vol 1, Ch 6.5.3 |
| Multi-curve risk decomposition: perturbations to basis swap spreads define basis risk | Andersen Vol 1, Ch 6.5.3 |
| TED spread definition and "spread of spreads" terminology | Tuckman Ch 17 |
| Spread of spreads trade: "designed not to express an opinion about the absolute level of TED spreads" | Tuckman Ch 17 |
| "Without the DV01 hedge, the net position in Eurodollar futures contracts would not be zero" | Tuckman Ch 17 |
| FNMA crowded trade: "forced liquidations that, in turn, widened the spreads even further" | Tuckman Ch 18 |
| Forward spread of spreads trade mechanics and P&L | Tuckman Ch 18, Appendix 18A |
| Mortgage hedging requires ~$30 billion swap adjustment per 25bp rate move | Tuckman Ch 21 |
| Mortgage hedging creates swap spread directionality (narrow in rallies, widen in selloffs) | Tuckman Ch 21 |
| "If the net basis of any bond is near zero, then the quality option embedded in the contract is nearly worthless" | Tuckman Ch 20 |
| Net basis of a bond close to CTD "behaves like a straddle on rates or prices" | Tuckman Ch 20 |
| CDS-bond basis driven by funding, delivery optionality, accrued premium treatment | O'Kane Ch 5.6 |

### (B) Reasoned Inference (Derived from A)

- **Three-layer decomposition framework (level/spread/residual)**: Synthesized from Tuckman's basis P&L analysis and Andersen's multi-curve risk factor decomposition. The framework organizes exposures logically based on how hedges are constructed.

- **"What you're long/short" articulation for each trade type**: Derived from the P&L formulas and economic logic. If P&L = change in net basis, then you are "long" whatever causes net basis to increase.

- **Hidden risks enumeration**: Categorized from various Tuckman discussions of funding, repo specialness, convexity, and convergence. Organized for pedagogical clarity rather than lifted verbatim.

- **Execution sizing principles**: Inferred from case study mechanics and the FNMA crowded trade example. The lesson about drawdown tolerance follows from the documented path behavior.

### (C) Flagged Uncertainties

- **Specific market conventions for modern SOFR basis swaps**: The transition from LIBOR to SOFR has created evolving conventions. The Andersen source predates the transition; current basis swap quotation conventions may differ in detail. *I'm not sure* about exact current market conventions without more recent practitioner sources.

- **Exact DV01 matching methodology across desks**: Different trading desks may use different conventions (forward DV01 vs. spot DV01, bump size) for sizing basis trades. The general principle is verified; desk-specific implementation varies.

- **Current regulatory capital treatment**: Bank capital rules affecting swap vs. Treasury positions have evolved significantly since the sources were written. Specifics require current regulatory documents. *I'm not sure* about the exact quantitative impact without Basel III/IV implementation details.

- **Magnitude of mortgage hedging flows in current market**: The $30 billion figure from Tuckman Ch 21 uses 2001 data. The MBS market has grown substantially; current hedging flow estimates would differ but the *mechanism* is verified.

---
