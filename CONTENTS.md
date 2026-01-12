# Fixed Income: Practice and Theory

## Table of Contents

---

## Part I — Foundations: Time Value, Rates, and Curve Primitives

### Chapter 1 — Market Quoting, Calendars, and Cashflow "Plumbing"
- Clean vs dirty, accrual intuition (preview)
- Day count / compounding as "unit system" of rates
- Settlement lags, payment timing, stubs (preview)

### Chapter 2 — Time Value of Money (TVM) and Discount Factors
- PV of deterministic cashflows
- Discount factor curve as the primitive object
- Simple vs compounded vs continuous rates (conceptual)

### Chapter 3 — Zero Rates, Forward Rates, Par Rates: The Triangle
- Conversions among discount factors / zero rates / forwards
- Par coupon/par swap rate intuition
- Sanity checks: monotonicity, negative rates (conceptual)

### Chapter 4 — Money-Market Building Blocks (The Shortest Curve Points)
- Deposits/bills: how they pin short discount factors
- FRAs and short-dated forwards
- Link to futures as forward-rate instruments (preview)

---

## Part II — Bonds: Pricing, Yields, Spreads, and Relative Value

### Chapter 5 — Fixed-Rate Bond Pricing (Clean/Dirty, Accrual, Cashflow PV)
- Cashflow schedule → PV with a curve
- Accrued interest mechanics
- Price quotation conventions (conceptual)

### Chapter 6 — Yield-to-Maturity (YTM) and Yield-Based Risk
- What YTM "means" and what it hides
- Yield-based DV01 vs curve-based DV01 (preview of the distinction)

### Chapter 7 — Bond Return Decomposition: Carry, Rolldown, and Curve Moves
- Carry and rolldown intuition and measurement
- "Curve shift vs spread change" decomposition concept

### Chapter 8 — Spread Measures for Bonds
- Why we need multiple spread definitions (G/I/Z/OAS/ASW—definitions later, but taxonomy now)
- Relative value logic: bond vs curve vs swap

### Chapter 9 — Repo as the Bond Market's Funding Engine
- GC vs specials, repo rate intuition
- Specialness bounds and fails logic (why specials can't go "infinitely special")
- Haircuts, funding liquidity (conceptual)

### Chapter 10 — Treasury Market Microstructure for Relative Value
- Auction cycle, on-the-run/off-the-run intuition
- Liquidity premia and financing advantage (conceptual)

---

## Part III — Risk Measures: Duration, DV01/PV01, Convexity, Key Rates, Hedging

### Chapter 11 — DV01/PV01: Definitions, Computation, and "What's Being Bumped"
- DV01 as price sensitivity per 1bp move (explicitly define the rate measure)
- Practical bump design: parallel vs localized bumps (preview)

### Chapter 12 — Duration (Macaulay/Modified) and Mapping to DV01
- Duration as scaled DV01
- Units checks and "per 100 notional" conventions

### Chapter 13 — Convexity: Second-Order P&L and When DV01 Breaks
- Convexity intuition and sign
- Why convexity matters for big moves / option-like bonds

### Chapter 14 — Key-Rate DV01 and Bucket Exposures
- Why "parallel DV01 = 0" ≠ "no risk"
- Bucket exposures and hedging bucket-by-bucket (Eurodollar futures example in the sources)

### Chapter 15 — DV01 Hedging: Hedge Ratios, Risk Weights, and Practical Caveats
- DV01 hedge ratio formula and its interpretation
- When DV01 hedges fail: curve twist, basis, convexity

### Chapter 16 — Curve Hedging Beyond DV01: Twists, Butterflies, PCA Intuition
- Curve-shape risk taxonomy (level/slope/curvature)
- Regression/PCA hedge concepts (high-level)

---

## Part IV — Yield Curve Construction: Single-Curve to Multi-Curve (OIS + Basis + Cross-Currency)

### Chapter 17 — Curve Construction as an Inverse Problem (Bootstrapping + Interpolation)
- Why interpolation is unavoidable with finite instruments
- Bootstrapping logic (instruments → discount factors)
- "Locality" intuition: how a quote change perturbs the curve (preview)

### Chapter 18 — OIS Discounting Curve: Building the "Risk-Free-ish" Curve
- OIS as a par instrument; extracting zero rates (conceptual)
- OIS rates can define a zero curve (explicitly noted in sources)

### Chapter 19 — Projection Curves: LIBOR/SOFR Term Curves and Why We Need Them
- Projection vs discounting separation
- Multi-curve motivation post-crisis (explicitly noted)

### Chapter 20 — Tenor Basis: 1M vs 3M vs 6M and Basis Swap Logic
- What "tenor basis" means and why it exists
- Practical consequences for pricing and hedging

### Chapter 21 — Cross-Currency Curves: CIP, FX Forwards, and Cross-Currency Basis
- Covered interest parity logic (conceptual)
- FX forwards vs cross-currency basis swaps as curve constraints (sources discuss CRX basis swaps and arbitrage consistency)

### Chapter 22 — Curve Risk Management in a Multi-Curve World
- Par-point deltas / Jacobian mapping (conceptual)
- Controlling perturbation behavior (why curve construction choices matter)

---

## Part V — Futures and Swaps: Valuation, Hedging, and Basis

### Chapter 23 — Treasury Futures: CTD, Conversion Factor, Delivery Options (Overview)
- Futures pricing link to repo/funding
- CTD intuition and why delivery options matter (conceptual)

### Chapter 24 — STIR Futures and Convexity Adjustments (Overview)
- Futures vs forward differences
- Convexity adjustment intuition (what it corrects)

### Chapter 25 — Interest Rate Swaps: Mechanics, Valuation, and Curve Dependencies
- Swap as exchange of fixed vs floating legs
- Discounting curve vs projection curve roles

### Chapter 26 — Swap PV01 / DV01 and Hedging with Swaps
- PV01 computation concept
- Swap DV01 vs bond DV01 mapping (conceptual)

### Chapter 27 — Swap Spreads, Asset Swaps, and Swap-Curve Relative Value
- What swap spreads measure (conceptual)
- Asset swap spread as bond-vs-swap relative value anchor

### Chapter 28 — Basis Trades in Rates: OIS–IBOR Basis, Swap Spread RV, Curve RV
- "What you're long/short" decomposition:
  - Discounting basis
  - Projection basis
  - Liquidity/funding components

---

## Part VI — FX Markets and Cross-Currency Derivatives (Rates + FX Integration)

### Chapter 29 — FX Spot and Forwards: Pricing via Interest Differentials
- Forward points, carry, hedging implications
- Link to CIP and curve construction (connect to Ch. 21)

### Chapter 30 — FX Swaps and Cross-Currency Swaps: Structure and Valuation Dependencies
- FX swap as funding instrument
- Cross-currency swap as exchange of floating legs + basis

### Chapter 31 — Multi-Currency Risk: FX Delta + Rates Delta + Basis Risk
- Practical risk decomposition (what to hedge with what)
- Why curve choice matters for hedges (ties back to Part IV)

---

## Part VII — Counterparty/Credit Risk for Derivatives and Collateral Discounting

### Chapter 32 — Counterparty Exposure Basics: Netting, Collateral, Margin Timing
- Potential future exposure intuition
- Netting sets and collateral mechanics (conceptual)

### Chapter 33 — Collateral Discounting and "Why OIS Shows Up Everywhere"
- Linking CSA collateral rates to discounting practice (ties to OIS curve)
- Why uncollateralized trades need adjustments (CVA noted in sources)

### Chapter 34 — XVA Overview: CVA/DVA/FVA (Conceptual + Implementation Map)
- What each adjustment "prices"
- Inputs and practical pitfalls (conceptual)

---

## Part VIII — Credit Fundamentals: Default, Recovery, Survival, and Cash-Credit Analytics

### Chapter 35 — Default, Recovery, and Credit Events: Economic vs Contractual Reality
- Recovery as a market-implied object
- "Credit event" definitions (preview of CDS)

### Chapter 36 — Survival Probabilities and Hazard Rates
- Survival curve Q(t) and hazard h(t) as modeling primitives
- Intuition: spread ↔ default intensity (conceptual bridge)

### Chapter 37 — Cash Credit: Risky Bonds, Credit Spreads, and Credit DV01
- Pricing risky cashflows (conceptual)
- Credit DV01 / spread duration intuition (ties to hedging mindset)

---

## Part IX — Single-Name CDS: Contracts, Auctions, Pricing, Curve Construction, Hedging

### Chapter 38 — CDS Contract Mechanics: Legs, Coupons, and Accrual
- Premium leg vs protection leg
- Accrued premium on default (explicitly described)

### Chapter 39 — CDS Credit Events and Settlement Choices
- Physical vs cash settlement
- Why "cheapest-to-deliver" matters (conceptual)

### Chapter 40 — The CDS Auction Process (What It Does and Why It Exists)
- Two-stage auction mechanism (explicit example in sources)
- Determining recovery / final price for settlement (conceptual)

### Chapter 41 — CDS Pricing Model (Par Spread, Upfront, Risky PV01)
- PV(premium leg) = PV(protection leg) equilibrium logic
- Running vs upfront conventions (conceptual)

### Chapter 42 — Bootstrapping a CDS Survival Curve from Market Quotes
- Piecewise hazard/survival construction steps (conceptual workflow)
- Consistency checks (monotonicity, positivity)

### Chapter 43 — Risks in CDS and Hedging the Risks
- Spread risk (CS01), jump-to-default risk, recovery risk
- Hedging with bonds vs CDS; basis intuition

### Chapter 44 — CDS Relative Value and Trading Frameworks (Risk-First, Not "Tips")
- Curve trades (steepener/flattener in credit curve)
- Capital structure / recovery trades (conceptual)

---

## Part X — CDS Indices, Index Rolls, Intrinsic Spreads, and Structured Credit (CDO / Tranches)

### Chapter 45 — CDS Indices: Structure, Quoting, and Lifecycle
- Index coupon vs upfront concept
- Roll schedules and what "on-the-run index" means (conceptual)

### Chapter 46 — Intrinsic Index Spread and Index Basis
- Intrinsic spread as "bottom-up" single-name implied level
- Drivers of index basis (restructuring clause + liquidity/supply/demand explicitly noted)

### Chapter 47 — Hedging and RV in Indices
- Index vs single-name hedge logic
- Default events and how index risk evolves (conceptual)

### Chapter 48 — CDO / Tranche Products: What They Are (Product Map)
- Standard tranches defined by attachment/detachment (explicit)
- Waterfall intuition: who absorbs first loss, who is protected

### Chapter 49 — Tranche "Core Concepts": Loss Distribution, Expected Tranche Loss, PV
- Portfolio loss as the state variable
- Equity vs mezz vs senior intuition

### Chapter 50 — Correlation and Tranche Pricing Frameworks (Conceptual Map)
- Why tranche valuation is "correlation-sensitive"
- Gaussian copula / base correlation as market language (high-level)

### Chapter 51 — Tranche Risk: Tranche PV01, Correlation Risk, Jump-to-Default Clustering
- Risk decompositions for tranche books
- Common hedging instruments: index, single-names (conceptual)

### Chapter 52 — Credit Trading Strategies (Structured as Risk + Instrument + Hedge)
- Basis strategies: bond–CDS, single-name–index
- Roll/down and carry in CDS indices
- Correlation/RV framing for tranches

> *Chapter 52 discipline: define strategy → identify risk exposures → hedge set → failure modes.*

---

## Optional Quant Appendices
*Take after Part X, only if you want modeling depth*

### Appendix A1 — No-Arbitrage Pricing, Numeraires, and Measure Changes (Rates Toolkit)
- Why "drift restrictions" appear in term-structure models (bridge to HJM)

### Appendix A2 — Short-Rate Models (Vasicek/CIR/Hull–White): What They Fit and Why
- Fitting the initial term structure (exogenous term structure idea)

### Appendix A3 — HJM Framework Essentials
- Drift determined by volatility (explicit point)

### Appendix A4 — Market Models: LMM and Swap Market Model; Calibration Logic
- Compatibility with Black cap/swaption formulas is a key motivation (explicit)

### Appendix A5 — Numerical Methods Map: Trees, PDE/Finite Differences, Monte Carlo
- When each method is used; stability/accuracy intuition (conceptual)

### Appendix A6 — Credit Portfolio Modeling Beyond Base Correlation (Advanced)
- Factor models / copulas / dynamic intensity sketches (high-level)

---

## Notes on Scope

**Speculation (clearly labeled; minimal):**

Some operational details (e.g., the latest ISDA credit-event definitions, auction timing mechanics, or current standard coupons for specific index series) can evolve with documentation and market practice. The outline above assumes we will treat the core mechanics from the books as canonical, and explicitly flag/document any "current-market" conventions if/when you want the notes to be desk-accurate for a specific market and date.
