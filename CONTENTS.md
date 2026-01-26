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

---

## Agent Instructions for Chapter Improvement

**IMPORTANT:** The existing chapter content was created by someone knowledgeable about fixed income. The core technical content is sound. Your job is to **improve presentation**, not rewrite substance.

### The Improvement Philosophy

| DO | DO NOT |
|----|--------|
| Improve clarity and wording | Change the technical meaning |
| Transform bullet notes into flowing prose | Remove or replace core explanations |
| Add transitions and context | Invent new concepts or formulas |
| Extend with book-sourced detail | Add content from general knowledge |
| Embed source citations naturally | Remove existing content arbitrarily |
| Make engaging for beginners AND experts | Dumb down the material |

### The Improvement Workflow

For each chapter you improve, follow this sequence:

```
1. READ the existing chapter fully — understand what's there
2. READ CONTENTS.md entry — understand the chapter's scope
3. IDENTIFY notes-style sections — bullet lists, terse explanations
4. SEARCH books/ deeply — find supporting detail, quotes, examples
5. REWRITE for flow — transform to textbook prose, keep core content
6. EXTEND where thin — add book-sourced context, never invent
7. VERIFY correctness — check formulas, calculations, citations
8. REVIEW for duplication — ensure no overlap with other chapters
```

### What "Transform to Textbook Style" Means

**Before (notes style):**
```markdown
## DV01
- Price sensitivity per 1bp
- DV01 = -dP/dy × 0.0001
- Used for hedging
```

**After (textbook style):**
```markdown
## 11.1 DV01: Price Sensitivity per Basis Point

When a trader says "I'm long 50k DV01," they mean their position gains
approximately $50,000 for every basis point drop in rates. This is the
language of interest rate risk.

Tuckman defines DV01 as "the change in price for a one-basis-point
decline in rates." More precisely:

$$\boxed{\text{DV01} = -\frac{\partial P}{\partial y} \times 0.0001}$$

The negative sign ensures DV01 is positive for typical bonds (which lose
value when rates rise). The 0.0001 factor converts from "per unit rate"
to "per basis point."

**Why this matters for hedging:** If you hold a bond with DV01 of $500
and want to hedge with a bond whose DV01 is $250, you need twice the
notional of the hedge...
```

**Key transformations:**
- Bullet → flowing prose
- Formula alone → formula with context and intuition
- Terse statement → explanation of "why it matters"
- No citation → embedded source attribution

### How to Extend Content

When a section feels thin, search books/ for:

1. **Concrete examples** — real numbers, typical market values
2. **The "why"** — reasoning behind conventions or formulas
3. **Common pitfalls** — what practitioners get wrong
4. **Edge cases** — what happens at boundaries (zero rates, defaults, etc.)
5. **Historical context** — why conventions evolved this way

**Example extension search:**
```
Grep "DV01" books/05_advanced/fixed_income/fixed_income_securities_tuckman.md
Grep "duration" books/02_portfolio_risk/portfolio_risk/options_futures_and_other_derivatives.md
Grep "price sensitivity" books/05_advanced/fixed_income/interest_rate_modeling.md
```

Read the matches. Quote relevant passages. Weave them into the narrative.

### Chapter-by-Chapter Notes

Below are specific notes for improving each chapter. Use these alongside the general workflow.

---

#### Part I — Foundations

**Chapter 1 (Market Quoting, Calendars, Plumbing)** — *COMPLETED as reference*
- Use as the style template for all other chapters
- Note the flowing introduction, numbered sections, embedded citations

**Chapter 2 (Time Value, Discount Factors)**
- Core content: discount factors as the primitive, PV of cashflows
- Extend: Add intuitive "what is a discount factor" explanation
- Search: "law of one price", "replication", "discount factor" in Tuckman, Hull
- Watch: Don't duplicate day count/compounding detail (Ch 1 owns that)

**Chapter 3 (Zero/Forward/Par Rates)**
- Core content: The rate triangle and conversions
- Extend: Add worked examples with specific numbers
- Search: "spot rate", "forward rate", "par rate", "instantaneous forward" across all books
- Watch: Don't derive bootstrapping algorithm (Ch 17 owns that)

**Chapter 4 (Money-Market Building Blocks)**
- Core content: How deposits/bills pin short discount factors
- Extend: Add typical market quotes, settlement conventions
- Search: "money market", "deposit", "T-bill", "FRA" in Hull, Tuckman
- Watch: Light preview of futures only (Ch 24 owns detail)

---

#### Part II — Bonds

**Chapter 5 (Bond Pricing)**
- Core content: Cashflow → PV, accrued interest formulas
- Extend: Multiple worked examples (Treasury, corporate)
- Search: "bond price", "invoice price", "accrued interest" comprehensively
- Watch: Chapter 1 previews clean/dirty; this chapter owns the formulas

**Chapter 6 (YTM and Yield Risk)**
- Core content: What YTM means and hides
- Extend: IRR interpretation, reinvestment assumption critique
- Search: "yield to maturity", "internal rate of return", "yield curve assumption"
- Watch: Yield-based DV01 is preview; Ch 11 owns DV01 fully

**Chapter 7 (Carry, Rolldown)**
- Core content: Return decomposition
- Extend: Numerical examples of carry calculation
- Search: "carry", "roll down", "return decomposition", "theta" in Tuckman

**Chapter 8 (Spread Measures)**
- Core content: G/I/Z/OAS/ASW taxonomy
- Extend: When to use which spread, relative value logic
- Search: "G-spread", "I-spread", "Z-spread", "OAS", "asset swap spread" across credit books

**Chapter 9 (Repo)**
- Core content: GC vs specials, funding mechanics
- Extend: Why specials exist, haircut intuition
- Search: "repo", "repurchase", "general collateral", "specials", "fails" in Tuckman

**Chapter 10 (Treasury Microstructure)**
- Core content: Auction cycle, on/off-the-run
- Extend: Liquidity premium intuition, financing advantage
- Search: "on-the-run", "off-the-run", "auction", "when-issued" in Tuckman

---

#### Part III — Risk Measures

**Chapter 11 (DV01/PV01)**
- Core content: Sensitivity per basis point, bump design
- Extend: Parallel vs localized bumps, "what's being bumped"
- Search: "DV01", "PV01", "dollar duration", "basis point value" comprehensively

**Chapter 12 (Duration)**
- Core content: Macaulay/modified, relation to DV01
- Extend: Units checks, "per 100 notional" conventions
- Search: "Macaulay duration", "modified duration", "duration" in Tuckman, Hull

**Chapter 13 (Convexity)**
- Core content: Second-order P&L, when DV01 breaks
- Extend: Sign intuition, callable bond convexity
- Search: "convexity", "gamma", "second derivative" in rates books

**Chapter 14 (Key-Rate DV01)**
- Core content: Why parallel DV01=0 ≠ no risk
- Extend: Bucket exposures, key rate methodology
- Search: "key rate", "bucket", "partial DV01", "KR01" in Tuckman, Andersen

**Chapter 15 (DV01 Hedging)**
- Core content: Hedge ratio formula
- Extend: When DV01 hedges fail (twist, basis, convexity)
- Search: "hedge ratio", "duration hedge", "basis risk"

**Chapter 16 (Curve Hedging)**
- Core content: Level/slope/curvature, PCA intuition
- Extend: Butterfly trades, regression hedging
- Search: "PCA", "principal component", "butterfly", "curve risk"

---

#### Part IV — Curve Construction

**Chapter 17 (Bootstrapping)**
- Core content: Inverse problem, interpolation necessity
- Extend: Spline methods, locality intuition
- Search: "bootstrap", "interpolation", "spline", "cubic" in Andersen, Hull

**Chapter 18 (OIS Curve)**
- Core content: OIS as risk-free-ish, extracting zeros
- Extend: Why OIS for discounting, post-crisis motivation
- Search: "OIS", "overnight index swap", "fed funds", "SOFR" comprehensively

**Chapter 19 (Projection Curves)**
- Core content: Projection vs discounting separation
- Extend: Multi-curve framework motivation
- Search: "projection curve", "LIBOR curve", "multi-curve", "basis"

**Chapter 20 (Tenor Basis)**
- Core content: 1M vs 3M vs 6M, basis swap logic
- Extend: Why tenor basis exists, practical consequences
- Search: "tenor basis", "basis swap", "1M 3M basis" in Andersen

**Chapter 21 (Cross-Currency)**
- Core content: CIP, FX forwards, xccy basis
- Extend: Arbitrage consistency, curve constraints
- Search: "covered interest parity", "cross-currency", "xccy basis", "FX forward"

**Chapter 22 (Multi-Curve Risk)**
- Core content: Jacobian, par-point deltas
- Extend: Why curve construction choices matter for risk
- Search: "Jacobian", "par-point", "curve sensitivity", "perturbation"

---

#### Part V — Futures and Swaps

**Chapter 23 (Treasury Futures)**
- Core content: CTD, conversion factor, delivery options
- Extend: Repo/funding link, basis intuition
- Search: "CTD", "cheapest to deliver", "conversion factor", "delivery option"

**Chapter 24 (STIR Futures)**
- Core content: Futures vs forward, convexity adjustment
- Extend: Why adjustment needed, magnitude intuition
- Search: "Eurodollar", "STIR", "convexity adjustment", "futures forward"

**Chapter 25 (IRS Mechanics)**
- Core content: Fixed vs floating legs, valuation
- Extend: Curve dependencies, CSA considerations
- Search: "interest rate swap", "fixed leg", "floating leg", "swap valuation"

**Chapter 26 (Swap PV01/DV01)**
- Core content: Swap sensitivity, annuity factor
- Extend: Swap vs bond DV01 comparison
- Search: "swap DV01", "swap PV01", "annuity", "swap sensitivity"

**Chapter 27 (Swap Spreads)**
- Core content: What swap spreads measure
- Extend: Asset swap spread as RV anchor
- Search: "swap spread", "asset swap", "Libor-Treasury spread"

**Chapter 28 (Basis Trades)**
- Core content: OIS-IBOR basis, RV decomposition
- Extend: "What you're long/short" framework
- Search: "basis trade", "OIS LIBOR basis", "funding basis"

---

#### Part VI — FX and Cross-Currency

**Chapter 29 (FX Forwards)**
- Core content: Forward points, carry
- Extend: CIP link, hedging implications
- Search: "FX forward", "forward points", "interest rate differential"

**Chapter 30 (Cross-Currency Swaps)**
- Core content: Structure, valuation dependencies
- Extend: FX swap vs xccy swap distinction
- Search: "cross-currency swap", "FX swap", "basis swap"

**Chapter 31 (Multi-Currency Risk)**
- Core content: FX + rates + basis decomposition
- Extend: Practical hedge instrument choices
- Search: "FX delta", "rates delta", "cross-gamma", "multi-currency"

---

#### Part VII — Counterparty Risk

**Chapter 32 (Counterparty Exposure)**
- Core content: PFE, netting, margin timing
- Extend: Exposure profile intuition
- Search: "PFE", "potential future exposure", "netting", "collateral"

**Chapter 33 (Collateral Discounting)**
- Core content: CSA rates → discounting, OIS everywhere
- Extend: Uncollateralized trade adjustments
- Search: "CSA", "collateral", "OIS discounting", "credit support annex"

**Chapter 34 (XVA Overview)**
- Core content: What each adjustment prices
- Extend: Inputs, practical pitfalls
- Search: "CVA", "DVA", "FVA", "XVA", "valuation adjustment"

---

#### Part VIII — Credit Fundamentals

**Chapter 35 (Default and Recovery)**
- Core content: Economic vs contractual reality
- Extend: Recovery as market-implied, credit event definitions
- Search: "default", "recovery rate", "credit event", "LGD"

**Chapter 36 (Survival Probabilities)**
- Core content: Q(t), hazard rates as primitives
- Extend: Spread ↔ intensity bridge
- Search: "survival probability", "hazard rate", "default intensity"

**Chapter 37 (Cash Credit)**
- Core content: Risky bond pricing, credit DV01
- Extend: Spread duration intuition
- Search: "risky bond", "credit spread", "spread duration", "credit DV01"

---

#### Part IX — CDS

**Chapter 38 (CDS Mechanics)**
- Core content: Premium vs protection legs
- Extend: Accrued premium on default
- Search: "CDS", "premium leg", "protection leg", "credit default swap"

**Chapter 39 (Credit Events)**
- Core content: Physical vs cash settlement
- Extend: CTD in CDS context
- Search: "credit event", "restructuring", "bankruptcy", "failure to pay"

**Chapter 40 (CDS Auction)**
- Core content: Two-stage mechanism
- Extend: Recovery determination process
- Search: "CDS auction", "credit event auction", "recovery determination"

**Chapter 41 (CDS Pricing)**
- Core content: Par spread, upfront, risky PV01
- Extend: PV equilibrium logic
- Search: "CDS pricing", "par spread", "upfront", "risky annuity"

**Chapter 42 (Survival Curve Bootstrap)**
- Core content: Piecewise hazard construction
- Extend: Consistency checks
- Search: "survival curve", "hazard rate bootstrap", "CDS bootstrap"

**Chapter 43 (CDS Risks)**
- Core content: CS01, JTD, recovery risk
- Extend: Bond vs CDS hedging
- Search: "CS01", "jump to default", "recovery risk", "CDS hedge"

**Chapter 44 (CDS Relative Value)**
- Core content: Curve trades, capital structure
- Extend: Steepener/flattener mechanics
- Search: "CDS curve trade", "credit curve", "steepener", "flattener"

---

#### Part X — Indices and Tranches

**Chapter 45 (CDS Indices)**
- Core content: Index coupon, upfront, roll
- Extend: CDX vs iTraxx differences
- Search: "CDX", "iTraxx", "credit index", "index roll"

**Chapter 46 (Intrinsic Spread)**
- Core content: Bottom-up implied level
- Extend: Index basis drivers
- Search: "intrinsic spread", "index basis", "skew"

**Chapter 47 (Index Hedging)**
- Core content: Index vs single-name logic
- Extend: Default impact on index
- Search: "index hedge", "single-name hedge", "index default"

**Chapter 48 (CDO/Tranches)**
- Core content: Attachment/detachment, waterfall
- Extend: Standard tranche definitions
- Search: "CDO", "tranche", "attachment", "detachment", "waterfall"

**Chapter 49 (Tranche Concepts)**
- Core content: Loss distribution, expected tranche loss
- Extend: Equity/mezz/senior intuition
- Search: "tranche loss", "expected loss", "tranche PV"

**Chapter 50 (Correlation)**
- Core content: Why tranches are correlation-sensitive
- Extend: Gaussian copula as market language
- Search: "base correlation", "Gaussian copula", "correlation smile"

**Chapter 51 (Tranche Risk)**
- Core content: Tranche PV01, correlation risk
- Extend: JTD clustering
- Search: "tranche delta", "correlation risk", "tranche sensitivity"

**Chapter 52 (Credit Strategies)**
- Core content: Basis, roll, correlation RV
- Extend: Risk + instrument + hedge framework
- Search: "basis strategy", "credit RV", "correlation trade"

---

#### Quant Appendices

**Appendix A1-A6**: These are more technical. Extend with:
- Precise mathematical statements from Brigo & Mercurio, Andersen
- Derivation steps (not just results)
- Connection to practical calibration/implementation
