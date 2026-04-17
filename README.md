# Fixed Income: Practice and Theory

*A practitioner-oriented textbook on how fixed income products are quoted, priced, risk-managed, and traded — from a Treasury quote of "99-16+" to the multi-curve machinery behind swaps, CDS, and tranches.*

---

A trader says a Treasury is "99-16+". A salesperson quotes a 10-year swap at 3.25%. A risk report shows DV01 of fifty thousand dollars. None of those are just numbers. They are **protocols** — compact encodings of day-count conventions, settlement lags, discounting choices, and sign conventions that determine what a trade actually pays and what "moving rates by one basis point" actually means.

Fixed income is often taught as a wall of such conventions. Treated that way, they look like trivia. Treated correctly, they are the subject: the interface between a market quote and real cash movements, P&L, and risk. This book is built around making that interface explicit.

**The book is free, open, and written in Markdown.** It contains **52 chapters and 6 quant appendices**. Chapters live under [`chapters/`](chapters/). Start wherever you like, or follow one of the curated paths in [`LEARNING_PATHS.md`](LEARNING_PATHS.md).

---

## The Core Idea

The book treats fixed income as a single pipeline of concrete objects:

> **quote → conventions → schedule → cashflows → curve(s) → PV → risk → hedges → P&L explain**

Every chapter reinforces one or more links in this chain. If you have ever seen two systems disagree on DV01, a hedge that "should be flat" still bleed money, or a clean/dirty mix-up create a reconciliation break, the cause was almost always an implicit assumption in one link. The chapters aim to make those assumptions explicit, with units, sign conventions, and sanity checks.

## What Makes This Book Different

- **Conventions are the subject, not the footnote.** Rate quotes are treated as *quote objects* with day-count, compounding, calendars, and settlement rules — not as numbers in a spreadsheet.
- **Curves are the primitive.** Discount factors come first; yields and par rates are derived summaries with specific failure modes.
- **Risk measures are fully specified.** Every DV01, PV01, CS01 is tied to a bump object, a bump size, units per notional, and a sign convention — so risk numbers are comparable across desks and actually hedgeable.
- **Desk reality and pitfalls get their own callouts.** Common breaks — clean/dirty mixing, forward ≠ forecast, multi-curve telescoping, CTD switch risk, "fully collateralized" ≠ zero exposure — are documented alongside the theory.
- **Worked numeric examples in every chapter.** Theory is grounded in at least one concrete calculation you can step through by hand.
- **Evidence-first.** Non-trivial conventions, formulas, and market practices are supported by references at the end of each chapter.

## Who This Book Is For

- **New analysts and associates** joining rates, credit, or macro desks.
- **Risk, product control, and operations staff** who need the "why" behind P&L attribution and reconciliation breaks.
- **Software engineers** building pricing and risk infrastructure who require convention-level correctness.
- **Graduate students and self-learners** with a basic finance background who want practical fixed income fluency before (or alongside) a modeling course.

Prerequisites vary by chapter. You can start at [Chapter 1](chapters/chapter_01_market_quoting_calendars_cashflow_plumbing.md) with no prior fixed income background. The optional quant appendices (A1–A6) assume comfort with stochastic calculus.

## What You Will Learn

By the end of the core chapters, you should be able to:

- Decode quote formats — 32nds, discount-yield bills, swap par rates, CDS spread and upfront conventions — into precise quote objects.
- Build schedules and cashflows with explicit calendars, settlement, and accrual rules, and compute invoice amounts from clean prices.
- Price bonds and swaps from discount factors and curve-based PV, and understand what yield-to-maturity hides.
- Speak spread correctly — G-spread, I-spread, Z-spread, OAS, asset-swap — and know which spread answers which question.
- Compute and interpret DV01, PV01, duration, convexity, and key-rate DV01 with full specification of what is being bumped and with what sign.
- Bootstrap and interrogate curves, including OIS discounting, projection curves, tenor basis, and cross-currency constraints — and understand why interpolation is a modeling choice with risk consequences.
- Connect trading language to economics: carry and rolldown vs MTM, repo specialness, CTD and implied repo, rates and credit basis trades.
- Reason about counterparty exposure, collateral mechanics, and XVA inputs and failure modes.
- Build credit intuition from default and recovery through hazard rates, CDS mechanics, index basis, and tranche correlation risk.

---

## Learning Paths

Reading linearly from Chapter 1 is the canonical approach and works well. Most readers, however, come to the book with a specific goal — onboarding to a desk, refreshing a topic, preparing for an interview. For those readers, [`LEARNING_PATHS.md`](LEARNING_PATHS.md) organizes the material into **seven curated paths**:

| Path | Goal |
|---|---|
| [Path 1](LEARNING_PATHS.md#path-1--core-fixed-income-recommended-start) | Core Fixed Income (Recommended Start) |
| [Path 2](LEARNING_PATHS.md#path-2--rates-desk-curves-and-swaps) | Rates Desk — Curves and Swaps |
| [Path 3](LEARNING_PATHS.md#path-3--risk-management-and-hedging) | Risk Management and Hedging |
| [Path 4](LEARNING_PATHS.md#path-4--credit-and-structured-credit) | Credit and Structured Credit |
| [Path 5](LEARNING_PATHS.md#path-5--counterparty-risk-and-xva) | Counterparty Risk and XVA |
| [Path 6](LEARNING_PATHS.md#path-6--quant-modeling-depth) | Quant Modeling Depth |
| [Path 7](LEARNING_PATHS.md#path-7--fx-and-cross-currency) | FX and Cross-Currency |

Each path lists prerequisites, a reading sequence, and what you should be able to do after completing it.

---

## Table of Contents

The book is organized into ten parts plus six optional quant appendices.

### Part I — Foundations: Time Value, Rates, and Curve Primitives

- [Chapter 1 — Market Quoting, Calendars, and Cashflow Plumbing](chapters/chapter_01_market_quoting_calendars_cashflow_plumbing.md)
- [Chapter 2 — Time Value of Money, Discount Factors, and Replication](chapters/chapter_02_time_value_discount_factors_replication.md)
- [Chapter 3 — Zero Rates, Forward Rates, Par Rates — The Triangle](chapters/chapter_03_zero_forward_par_rates_triangle.md)
- [Chapter 4 — Money-Market Building Blocks (The Shortest Curve Points)](chapters/chapter_04_money_market_building_blocks.md)

### Part II — Bonds: Pricing, Yields, Spreads, and Relative Value

- [Chapter 5 — Fixed-Rate Bond Pricing](chapters/chapter_05_fixed_rate_bond_pricing.md)
- [Chapter 6 — Yield-to-Maturity and Yield-Based Risk](chapters/chapter_06_ytm_yield_based_risk.md)
- [Chapter 7 — Bond Return Decomposition: Carry, Rolldown, Curve Moves, and Spread Changes](chapters/chapter_07_bond_return_decomposition.md)
- [Chapter 8 — Spreads 101: G-spread, I-spread, Z-spread, OAS, and What Spread Are We Talking About](chapters/chapter_08_spreads_101.md)
- [Chapter 9 — Repo: The Bond Market's Funding Engine](chapters/chapter_09_repo_funding_engine.md)
- [Chapter 10 — Treasury Market Microstructure and Relative Value](chapters/chapter_10_treasury_microstructure_relative_value.md)

### Part III — Risk Measures: Duration, DV01/PV01, Convexity, Key Rates, Hedging

- [Chapter 11 — DV01 and PV01: Definitions, Computation, and What's Being Bumped](chapters/chapter_11_dv01_pv01_definitions_computation.md)
- [Chapter 12 — Duration: Macaulay, Modified, and the Connection to DV01](chapters/chapter_12_duration_macaulay_modified_dv01.md)
- [Chapter 13 — Convexity: Second-Order P&L and When DV01 Breaks](chapters/chapter_13_convexity.md)
- [Chapter 14 — Key-Rate DV01 and Bucket Exposures](chapters/chapter_14_key_rate_dv01_bucket_exposures.md)
- [Chapter 15 — DV01 Hedging: Hedge Ratios, Risk Weights, and Practical Caveats](chapters/chapter_15_dv01_hedging.md)
- [Chapter 16 — Curve Hedging Beyond DV01: Twists, Butterflies, and PCA](chapters/chapter_16_curve_hedging_twists_butterflies_pca.md)

### Part IV — Yield Curve Construction: Single-Curve to Multi-Curve

- [Chapter 17 — Curve Construction: Bootstrapping, Interpolation, and the Spline Zoo](chapters/chapter_17_curve_construction_bootstrapping_interpolation.md)
- [Chapter 18 — OIS Discounting Curve: Building the Risk-Free-ish Curve](chapters/chapter_18_ois_discounting_curve.md)
- [Chapter 19 — Projection Curves: LIBOR, SOFR, and the Multi-Curve Framework](chapters/chapter_19_projection_curves_libor_sofr_multi_curve.md)
- [Chapter 20 — Tenor Basis: 1M vs 3M vs 6M and Basis Swap Logic](chapters/chapter_20_tenor_basis.md)
- [Chapter 21 — Cross-Currency Curves: CIP, FX Forwards, and Cross-Currency Basis](chapters/chapter_21_cross_currency_curves.md)
- [Chapter 22 — Multi-Curve Risk Management: Par-Point Deltas, Jacobians, and Controlled Perturbations](chapters/chapter_22_multi_curve_risk_jacobians.md)

### Part V — Futures and Swaps: Valuation, Hedging, and Basis

- [Chapter 23 — Treasury Futures: CTD, Conversion Factors, Delivery Options, and the Link to Repo](chapters/chapter_23_treasury_futures.md)
- [Chapter 24 — STIR Futures and Convexity Adjustments](chapters/chapter_24_stir_futures_convexity_adjustments.md)
- [Chapter 25 — Interest Rate Swaps: Mechanics and Valuation](chapters/chapter_25_interest_rate_swaps_mechanics_valuation.md)
- [Chapter 26 — Swap PV01, DV01, and Hedging with Swaps](chapters/chapter_26_swap_pv01_dv01_hedging.md)
- [Chapter 27 — Swap Spreads, Asset Swaps, and Swap-Curve Relative Value](chapters/chapter_27_swap_spreads_asset_swaps_swap_curve_rv.md)
- [Chapter 28 — Basis Trades in Rates: OIS-IBOR Basis, Futures Basis, Swap-Spread RV, Curve RV](chapters/chapter_28_basis_trades.md)

### Part VI — FX Markets and Cross-Currency Derivatives

- [Chapter 29 — FX Spot and Forwards: Pricing via Interest Differentials](chapters/chapter_29_fx_spot_forwards.md)
- [Chapter 30 — FX Swaps and Cross-Currency Swaps: Structure and Valuation Dependencies](chapters/chapter_30_fx_swaps_cross_currency_swaps.md)
- [Chapter 31 — Multi-Currency Risk: FX Delta, Rates Delta, Cross-Gamma, and Aggregation](chapters/chapter_31_multi_currency_risk.md)

### Part VII — Counterparty Risk, Collateral Discounting, and XVA

- [Chapter 32 — Counterparty Exposure Basics: Netting, Collateral, and Margin Timing](chapters/chapter_32_counterparty_exposure_basics.md)
- [Chapter 33 — Collateral Discounting and OIS](chapters/chapter_33_collateral_discounting_ois.md)
- [Chapter 34 — XVA Overview: CVA, DVA, FVA, and the New Derivatives Valuation Landscape](chapters/chapter_34_xva_overview.md)

### Part VIII — Credit Fundamentals

- [Chapter 35 — Default, Recovery, and Credit Events: Economic vs Contractual Reality](chapters/chapter_35_default_recovery_credit_events.md)
- [Chapter 36 — Survival Probabilities and Hazard Rates](chapters/chapter_36_survival_probabilities_hazard_rates.md)
- [Chapter 37 — Cash Credit: Risky Bonds, Credit Spreads, and CS01](chapters/chapter_37_cash_credit_risky_bonds_spreads_cs01.md)

### Part IX — Credit Derivatives: Single-Name CDS and Indices

- [Chapter 38 — CDS Contract Mechanics](chapters/chapter_38_cds_contract_mechanics.md)
- [Chapter 39 — CDS Credit Events and Settlement](chapters/chapter_39_cds_credit_events_settlement.md)
- [Chapter 40 — The CDS Auction Process](chapters/chapter_40_cds_auction_process.md)
- [Chapter 41 — CDS Indices: Mechanics, Coupons, Rolls](chapters/chapter_41_cds_indices_mechanics_coupons_rolls.md)
- [Chapter 42 — Bootstrapping a CDS Survival Curve from Market Quotes](chapters/chapter_42_bootstrapping_cds_survival_curve.md)
- [Chapter 43 — Risks in CDS and Hedging Strategies](chapters/chapter_43_cds_risks_hedging.md)
- [Chapter 44 — CDS Relative Value Trading Frameworks](chapters/chapter_44_cds_relative_value_trading_frameworks.md)
- [Chapter 45 — CDS Indices: Structure, Quoting, and Lifecycle](chapters/chapter_45_cds_indices_structure_quoting_lifecycle.md)
- [Chapter 46 — Intrinsic Index Spread and Index Basis](chapters/chapter_46_intrinsic_index_spread_and_index_basis.md)
- [Chapter 47 — Hedging and Relative Value in CDS Indices](chapters/chapter_47_hedging_relative_value_cds_indices.md)

### Part X — Structured Credit: Tranches, Correlation, and Strategies

- [Chapter 48 — CDO and Tranche Products: The Product Map](chapters/chapter_48_cdo_tranche_products_product_map.md)
- [Chapter 49 — Tranche Core Concepts: Expected Tranche Loss and Present Value](chapters/chapter_49_tranche_core_concepts_etl_pv.md)
- [Chapter 50 — Correlation and Tranche Pricing Frameworks](chapters/chapter_50_correlation_tranche_pricing_frameworks.md)
- [Chapter 51 — Tranche Risk: Tranche PV01, Correlation Risk, and Jump-to-Default Clustering](chapters/chapter_51_tranche_risk.md)
- [Chapter 52 — Credit Trading Strategies: Risk, Instrument, and Hedge](chapters/chapter_52_credit_trading_strategies.md)

### Optional Quant Appendices

*Take after Part X, only if you want modeling depth.*

- [Appendix A1 — No-Arbitrage Pricing, Numeraires, and Measure Changes](chapters/appendix_a1_no_arbitrage_numeraires_measure_changes.md)
- [Appendix A2 — Short-Rate Models (Vasicek, CIR, Hull–White)](chapters/appendix_a2_short_rate_models.md)
- [Appendix A3 — HJM Framework Essentials](chapters/appendix_a3_hjm_framework_essentials.md)
- [Appendix A4 — Market Models: LMM and Swap Market Model; Calibration Logic](chapters/appendix_a4_market_models_lmm_swap_calibration.md)
- [Appendix A5 — Numerical Methods: Trees, PDE / Finite Differences, Monte Carlo, and Fourier Methods](chapters/appendix_a5_numerical_methods_trees_pde_monte_carlo.md)
- [Appendix A6 — Credit Portfolio Modeling Beyond Base Correlation](chapters/appendix_a6_credit_portfolio_modeling.md)

---

## Quick Problem Index

If you came here with a specific question, start at the chapter below.

| If you want to… | Start here |
|---|---|
| Convert Treasury quotes like `99-16+` into dollars and understand clean vs dirty settlement | [Ch. 1](chapters/chapter_01_market_quoting_calendars_cashflow_plumbing.md), [Ch. 5](chapters/chapter_05_fixed_rate_bond_pricing.md) |
| Understand T-bill discount-yield quotes (and why they are not an IRR) | [Ch. 1](chapters/chapter_01_market_quoting_calendars_cashflow_plumbing.md), [Ch. 4](chapters/chapter_04_money_market_building_blocks.md) |
| Understand the zero / forward / par triangle and why forwards are not forecasts | [Ch. 3](chapters/chapter_03_zero_forward_par_rates_triangle.md) |
| Decompose bond returns into carry, rolldown, curve moves, and spread changes | [Ch. 7](chapters/chapter_07_bond_return_decomposition.md) |
| Speak spread correctly (G / I / Z / OAS / ASW) and know which spread answers which question | [Ch. 8](chapters/chapter_08_spreads_101.md) |
| Understand repo, specialness, and how funding shows up in relative value | [Ch. 9](chapters/chapter_09_repo_funding_engine.md) |
| Understand DV01 / PV01 in a risk report (and why two DV01s can disagree) | [Ch. 11](chapters/chapter_11_dv01_pv01_definitions_computation.md) |
| Build curves via bootstrapping and interpolation, and see why curve choices affect hedges | [Ch. 17](chapters/chapter_17_curve_construction_bootstrapping_interpolation.md) |
| Work in a modern multi-curve setup (OIS discounting, projection curves, tenor and cross-currency basis) | [Ch. 18](chapters/chapter_18_ois_discounting_curve.md) – [Ch. 22](chapters/chapter_22_multi_curve_risk_jacobians.md) |
| Understand Treasury futures: CTD, conversion factors, invoice pricing, implied repo, CTD switch risk | [Ch. 23](chapters/chapter_23_treasury_futures.md) |
| Avoid the classic multi-curve swap-PV mistake (telescoping a float leg priced on a different curve) | [Ch. 25](chapters/chapter_25_interest_rate_swaps_mechanics_valuation.md) |
| Understand counterparty exposure and why daily VM still leaves MPOR / gap risk | [Ch. 32](chapters/chapter_32_counterparty_exposure_basics.md), [Ch. 34](chapters/chapter_34_xva_overview.md) |
| Learn CDS mechanics (premium accrual, settlement, auctions, upfront-plus-coupon quoting) | [Ch. 38](chapters/chapter_38_cds_contract_mechanics.md) – [Ch. 40](chapters/chapter_40_cds_auction_process.md) |
| Bootstrap a CDS survival curve and connect spreads to hazard rates | [Ch. 36](chapters/chapter_36_survival_probabilities_hazard_rates.md), [Ch. 42](chapters/chapter_42_bootstrapping_cds_survival_curve.md) |
| Build structured-credit intuition (attachment / detachment, tranche PV, correlation risk) | [Ch. 48](chapters/chapter_48_cdo_tranche_products_product_map.md) – [Ch. 51](chapters/chapter_51_tranche_risk.md) |

---

## How to Read This Repo

- **On GitHub:** browse the [table of contents](#table-of-contents) above and follow the chapter links. Math and tables render natively.
- **Locally:** clone the repository and open `chapters/*.md` in any Markdown editor that supports LaTeX (VS Code, Obsidian, Typora, etc.).
- **By goal:** pick a path from [`LEARNING_PATHS.md`](LEARNING_PATHS.md).
- **By problem:** use the [Quick Problem Index](#quick-problem-index) above.

Each chapter is self-contained and follows the same structure: introduction, learning objectives, numbered content sections, summary, and references. Most include worked numeric examples, desk-reality callouts, and a list of common pitfalls. Cross-chapter links point to prerequisites and follow-ons so you can skip around without getting lost.

## Contributions and Errata

Corrections, clarifications, and worked examples are welcome. The most valuable contributions are:

- Convention and sign-convention corrections (ideally with a dated authoritative source when the convention is market-dependent).
- Worked examples and sanity checks that make a tricky concept stick.
- Fixes for broken links, typos, or unclear explanations.

Please open an issue or pull request. For methodology and writing conventions, see [`AGENTS.md`](AGENTS.md).

## Disclaimer

This book is intended for education. It is not investment advice.

Market conventions differ across products, jurisdictions, venues, and time. Always confirm details against authoritative documentation — issuer terms, ISDA and ARRC publications, CCP and exchange rulebooks, central-bank and regulator documents — before relying on any of this material in a production system.

## License

The content of this repository is provided for educational use. See the repository settings for the governing license.
