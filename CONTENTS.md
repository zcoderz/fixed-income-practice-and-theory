# Fixed Income: Practice and Theory

## Table of Contents

---

## Part I — Foundations: Time Value, Rates, and Curve Primitives

### [Chapter 1 — Market Quoting, Calendars, and Cashflow Plumbing](chapters/chapter_01_market_quoting_calendars_cashflow_plumbing.md)
- Clean vs dirty, accrual intuition (preview)
- Day count / compounding as "unit system" of rates
- Settlement lags, payment timing, stubs (preview)

### [Chapter 2 — Time Value of Money, Discount Factors, and Replication](chapters/chapter_02_time_value_discount_factors_replication.md)
- PV of deterministic cashflows
- Discount factor curve as the primitive object
- Simple vs compounded vs continuous rates (conceptual)

### [Chapter 3 — Zero Rates, Forward Rates, Par Rates — The Triangle](chapters/chapter_03_zero_forward_par_rates_triangle.md)
- Conversions among discount factors / zero rates / forwards
- Par coupon/par swap rate intuition
- Sanity checks: monotonicity, negative rates (conceptual)

### [Chapter 4 — Money-Market Building Blocks (The Shortest Curve Points)](chapters/chapter_04_money_market_building_blocks.md)
- Deposits/bills: how they pin short discount factors
- FRAs and short-dated forwards
- Link to futures as forward-rate instruments (preview)

---

## Part II — Bonds: Pricing, Yields, Spreads, and Relative Value

### [Chapter 5 — Fixed-Rate Bond Pricing](chapters/chapter_05_fixed_rate_bond_pricing.md)
- Cashflow schedule → PV with a curve
- Accrued interest mechanics
- Price quotation conventions (conceptual)

### [Chapter 6 — Yield-to-Maturity and Yield-Based Risk](chapters/chapter_06_ytm_yield_based_risk.md)
- What YTM "means" and what it hides
- Yield-based DV01 vs curve-based DV01 (preview of the distinction)

### [Chapter 7 — Bond Return Decomposition — Carry, Rolldown, Curve Moves, and Spread Changes](chapters/chapter_07_bond_return_decomposition.md)
- Carry and rolldown intuition and measurement
- "Curve shift vs spread change" decomposition concept

### [Chapter 8 — Spreads 101 — G-spread, I-spread, Z-spread, OAS, and “What Spread Are We Talking About?”](chapters/chapter_08_spreads_101.md)
- Why we need multiple spread definitions (G/I/Z/OAS/ASW—definitions later, but taxonomy now)
- Relative value logic: bond vs curve vs swap

### [Chapter 9 — Repo — The Bond Market’s Funding Engine](chapters/chapter_09_repo_funding_engine.md)
- GC vs specials, repo rate intuition
- Specialness bounds and fails logic (why specials can't go "infinitely special")
- Haircuts, funding liquidity (conceptual)

### [Chapter 10 — Treasury Market Microstructure and Relative Value](chapters/chapter_10_treasury_microstructure_relative_value.md)
- Auction cycle, on-the-run/off-the-run intuition
- Liquidity premia and financing advantage (conceptual)

---

## Part III — Risk Measures: Duration, DV01/PV01, Convexity, Key Rates, Hedging

### [Chapter 11 — DV01/PV01 — Definitions, Computation, and “What’s Being Bumped”](chapters/chapter_11_dv01_pv01_definitions_computation.md)
- DV01 as price sensitivity per 1bp move (explicitly define the rate measure)
- Practical bump design: parallel vs localized bumps (preview)

### [Chapter 12 — Duration — Macaulay, Modified, and the Connection to DV01](chapters/chapter_12_duration_macaulay_modified_dv01.md)
- Duration as scaled DV01
- Units checks and "per 100 notional" conventions

### [Chapter 13 — Convexity — Second-Order P&L and When DV01 Breaks](chapters/chapter_13_convexity.md)
- Convexity intuition and sign
- Why convexity matters for big moves / option-like bonds

### [Chapter 14 — Key-Rate DV01 and Bucket Exposures](chapters/chapter_14_key_rate_dv01_bucket_exposures.md)
- Why "parallel DV01 = 0" ≠ "no risk"
- Bucket exposures and hedging bucket-by-bucket (Eurodollar futures example in the sources)

### [Chapter 15 — DV01 Hedging — Hedge Ratios, Risk Weights, and Practical Caveats](chapters/chapter_15_dv01_hedging.md)
- DV01 hedge ratio formula and its interpretation
- When DV01 hedges fail: curve twist, basis, convexity

### [Chapter 16 — Curve Hedging Beyond DV01 — Twists, Butterflies, and PCA](chapters/chapter_16_curve_hedging_twists_butterflies_pca.md)
- Curve-shape risk taxonomy (level/slope/curvature)
- Regression/PCA hedge concepts (high-level)

---

## Part IV — Yield Curve Construction: Single-Curve to Multi-Curve (OIS + Basis + Cross-Currency)

### [Chapter 17 — Curve Construction — Bootstrapping, Interpolation, and the Spline Zoo](chapters/chapter_17_curve_construction_bootstrapping_interpolation.md)
- Why interpolation is unavoidable with finite instruments
- Bootstrapping logic (instruments → discount factors)
- "Locality" intuition: how a quote change perturbs the curve (preview)

### [Chapter 18 — OIS Discounting Curve — Building the “Risk-Free-ish” Curve](chapters/chapter_18_ois_discounting_curve.md)
- OIS as a par instrument; extracting zero rates (conceptual)
- OIS rates can define a zero curve (explicitly noted in sources)

### [Chapter 19 — Projection Curves — LIBOR, SOFR, and the Multi-Curve Framework](chapters/chapter_19_projection_curves_libor_sofr_multi_curve.md)
- Projection vs discounting separation
- Multi-curve motivation post-crisis (explicitly noted)

### [Chapter 20 — Tenor Basis — 1M vs 3M vs 6M and Basis Swap Logic](chapters/chapter_20_tenor_basis.md)
- What "tenor basis" means and why it exists
- Practical consequences for pricing and hedging

### [Chapter 21 — Cross-Currency Curves — CIP, FX Forwards, and Cross-Currency Basis as Curve Constraints](chapters/chapter_21_cross_currency_curves.md)
- Covered interest parity logic (conceptual)
- FX forwards vs cross-currency basis swaps as curve constraints (sources discuss CRX basis swaps and arbitrage consistency)

### [Chapter 22 — Curve Risk Management in a Multi-Curve World — Par-Point Deltas, Jacobians, and Controlled Perturbations](chapters/chapter_22_multi_curve_risk_jacobians.md)
- Par-point deltas / Jacobian mapping (conceptual)
- Controlling perturbation behavior (why curve construction choices matter)

---

## Part V — Futures and Swaps: Valuation, Hedging, and Basis

### [Chapter 23 — Treasury Futures — CTD, Conversion Factors, Delivery Options, and the Link to Repo](chapters/chapter_23_treasury_futures.md)
- Futures pricing link to repo/funding
- CTD intuition and why delivery options matter (conceptual)

### [Chapter 24 — STIR Futures and Convexity Adjustments](chapters/chapter_24_stir_futures_convexity_adjustments.md)
- Futures vs forward differences
- Convexity adjustment intuition (what it corrects)

### [Chapter 25 — Interest Rate Swaps — Mechanics and Valuation](chapters/chapter_25_interest_rate_swaps_mechanics_valuation.md)
- Swap as exchange of fixed vs floating legs
- Discounting curve vs projection curve roles

### [Chapter 26 — Swap PV01, DV01, and Hedging with Swaps](chapters/chapter_26_swap_pv01_dv01_hedging.md)
- PV01 computation concept
- Swap DV01 vs bond DV01 mapping (conceptual)

### [Chapter 27 — Swap Spreads, Asset Swaps, and Swap-Curve Relative Value](chapters/chapter_27_swap_spreads_asset_swaps_swap_curve_rv.md)
- What swap spreads measure (conceptual)
- Asset swap spread as bond-vs-swap relative value anchor

### [Chapter 28 — Basis Trades in Rates — OIS-IBOR Basis, Treasury Futures Basis, Swap Spread RV, Curve RV](chapters/chapter_28_basis_trades.md)
- "What you're long/short" decomposition:
  - Discounting basis
  - Projection basis
  - Liquidity/funding components

---

## Part VI — FX Markets and Cross-Currency Derivatives (Rates + FX Integration)

### [Chapter 29 — FX Spot and Forwards — Pricing via Interest Differentials](chapters/chapter_29_fx_spot_forwards.md)
- Forward points, carry, hedging implications
- Link to CIP and curve construction (connect to Ch. 21)

### [Chapter 30 — FX Swaps and Cross-Currency Swaps — Structure and Valuation Dependencies](chapters/chapter_30_fx_swaps_cross_currency_swaps.md)
- FX swap as funding instrument
- Cross-currency swap as exchange of floating legs + basis

### [Chapter 31 — Multi-Currency Risk — FX Delta, Rates Delta, Cross-Gamma, and Risk Aggregation](chapters/chapter_31_multi_currency_risk.md)
- Practical risk decomposition (what to hedge with what)
- Why curve choice matters for hedges (ties back to Part IV)

---

## Part VII — Counterparty/Credit Risk for Derivatives and Collateral Discounting

### [Chapter 32 — Counterparty Exposure Basics — Netting, Collateral, and Margin Timing](chapters/chapter_32_counterparty_exposure_basics.md)
- Potential future exposure intuition
- Netting sets and collateral mechanics (conceptual)

### [Chapter 33 — Collateral Discounting and OIS](chapters/chapter_33_collateral_discounting_ois.md)
- Linking CSA collateral rates to discounting practice (ties to OIS curve)
- Why uncollateralized trades need adjustments (CVA noted in sources)

### [Chapter 34 — XVA Overview — CVA, DVA, FVA, and the New Derivatives Valuation Landscape](chapters/chapter_34_xva_overview.md)
- What each adjustment "prices"
- Inputs and practical pitfalls (conceptual)

---

## Part VIII — Credit Fundamentals: Default, Recovery, Survival, and Cash-Credit Analytics

### [Chapter 35 — Default, Recovery, and Credit Events — Economic vs Contractual Reality](chapters/chapter_35_default_recovery_credit_events.md)
- Recovery as a market-implied object
- "Credit event" definitions (preview of CDS)

### [Chapter 36 — Survival Probabilities and Hazard Rates](chapters/chapter_36_survival_probabilities_hazard_rates.md)
- Survival curve Q(t) and hazard h(t) as modeling primitives
- Intuition: spread ↔ default intensity (conceptual bridge)

### [Chapter 37 — Cash Credit — Risky Bonds, Credit Spreads, and CS01](chapters/chapter_37_cash_credit_risky_bonds_spreads_cs01.md)
- Pricing risky cashflows (conceptual)
- CS01 / credit DV01 (spread duration) intuition (ties to hedging mindset)

---

## Part IX — Credit Derivatives: Single-Name CDS and Indices

### [Chapter 38 — CDS Contract Mechanics](chapters/chapter_38_cds_contract_mechanics.md)
- Premium leg vs protection leg
- Accrued premium on default (explicitly described)
- Risky PV01 (RPV01) and the upfront-plus-coupon trading regime

### [Chapter 39 — CDS Credit Events and Settlement](chapters/chapter_39_cds_credit_events_settlement.md)
- Physical vs cash settlement
- Why "cheapest-to-deliver" matters (conceptual)

### [Chapter 40 — The CDS Auction Process — What It Does and Why It Exists](chapters/chapter_40_cds_auction_process.md)
- Two-stage auction mechanism (explicit example in sources)
- Determining recovery / final price for settlement (conceptual)

### [Chapter 41 — CDS Indices — Mechanics, Coupons, Rolls](chapters/chapter_41_cds_indices_mechanics_coupons_rolls.md)
- Index families, naming, and lifecycle (on-the-run vs off-the-run)
- Fixed coupon + upfront mechanics (spread vs price intuition for indices)
- Defaults, roll dynamics, and the link to intrinsic value / basis (preview)

### [Chapter 42 — Bootstrapping a CDS Survival Curve from Market Quotes](chapters/chapter_42_bootstrapping_cds_survival_curve.md)
- Piecewise hazard/survival construction steps (conceptual workflow)
- Consistency checks (monotonicity, positivity)

### [Chapter 43 — Risks in CDS and Hedging Strategies](chapters/chapter_43_cds_risks_hedging.md)
- Spread risk (CS01), jump-to-default risk, recovery risk
- Hedging with bonds vs CDS; basis intuition

### [Chapter 44 — CDS Relative Value Trading Frameworks](chapters/chapter_44_cds_relative_value_trading_frameworks.md)
- Curve trades (steepener/flattener in credit curve)
- Capital structure / recovery trades (conceptual)

### [Chapter 45 — CDS Indices — Structure, Quoting, and Lifecycle](chapters/chapter_45_cds_indices_structure_quoting_lifecycle.md)
- Index naming conventions and identifier decoding
- Quoting conventions (spread vs price) and lifecycle mechanics
- Default handling and intrinsic spread / basis (preview; developed in Chapter 46)

### [Chapter 46 — Intrinsic Index Spread and Index Basis](chapters/chapter_46_intrinsic_index_spread_and_index_basis.md)
- Intrinsic spread as "bottom-up" single-name implied level
- Drivers of index basis (restructuring clause + liquidity/supply/demand explicitly noted)

### [Chapter 47 — Hedging and Relative Value in CDS Indices](chapters/chapter_47_hedging_relative_value_cds_indices.md)
- Index vs single-name hedge logic
- Default events and how index risk evolves (conceptual)

---

## Part X — Structured Credit: CDO / Tranches, Correlation, and Strategies

### [Chapter 48 — CDO / Tranche Products — What They Are (Product Map)](chapters/chapter_48_cdo_tranche_products_product_map.md)
- Standard tranches defined by attachment/detachment (explicit)
- Waterfall intuition: who absorbs first loss, who is protected

### [Chapter 49 — Tranche Core Concepts — Expected Tranche Loss and Present Value](chapters/chapter_49_tranche_core_concepts_etl_pv.md)
- Portfolio loss as the state variable
- Equity vs mezz vs senior intuition

### [Chapter 50 — Correlation and Tranche Pricing Frameworks](chapters/chapter_50_correlation_tranche_pricing_frameworks.md)
- Why tranche valuation is "correlation-sensitive"
- Gaussian copula / base correlation as market language (high-level)

### [Chapter 51 — Tranche Risk — Tranche PV01, Correlation Risk, and Jump-to-Default Clustering](chapters/chapter_51_tranche_risk.md)
- Risk decompositions for tranche books
- Common hedging instruments: index, single-names (conceptual)

### [Chapter 52 — Credit Trading Strategies (Risk + Instrument + Hedge)](chapters/chapter_52_credit_trading_strategies.md)
- Basis strategies: bond–CDS, single-name–index
- Roll/down and carry in CDS indices
- Correlation/RV framing for tranches

---

## Optional Quant Appendices
*Take after Part X, only if you want modeling depth*

### [Appendix A1 — No-Arbitrage Pricing, Numeraires, and Measure Changes](chapters/appendix_a1_no_arbitrage_numeraires_measure_changes.md)
- Why "drift restrictions" appear in term-structure models (bridge to HJM)

### [Appendix A2 — Short-Rate Models (Vasicek / CIR / Hull–White)](chapters/appendix_a2_short_rate_models.md)
- Fitting the initial term structure (exogenous term structure idea)

### [Appendix A3 — HJM Framework Essentials](chapters/appendix_a3_hjm_framework_essentials.md)
- Drift determined by volatility (explicit point)

### [Appendix A4 — Market Models — LMM and Swap Market Model; Calibration Logic](chapters/appendix_a4_market_models_lmm_swap_calibration.md)
- Compatibility with Black cap/swaption formulas is a key motivation (explicit)

### [Appendix A5 — Numerical Methods — Trees, PDE / Finite Differences, Monte Carlo, and Fourier Methods](chapters/appendix_a5_numerical_methods_trees_pde_monte_carlo.md)
- When each method is used; stability/accuracy intuition (conceptual)

### [Appendix A6 — Credit Portfolio Modeling Beyond Base Correlation](chapters/appendix_a6_credit_portfolio_modeling.md)
- Factor models / copulas / dynamic intensity sketches (high-level)
