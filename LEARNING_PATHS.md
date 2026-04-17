# Learning Paths

*Fixed Income: Practice and Theory* spans 52 chapters plus 6 quant appendices. Reading linearly from Chapter 1 through Chapter 52 is the canonical order, but most readers don't need the entire book — or don't need it in that order.

This document offers **seven curated paths**, each a short sequence of chapters assembled around a specific goal: a first pass, a rates-desk onboarding, a risk-management refresher, a credit track, and so on. Paths overlap intentionally — concepts like DV01, curves, and discounting appear in several — so you can switch paths whenever your questions change.

For the complete chapter-by-chapter table of contents, see the [README](README.md#table-of-contents).

---

## How to Choose a Path

| If you are… | Start with |
|---|---|
| New to fixed income and want one coherent foundation | [Path 1 — Core Fixed Income](#path-1--core-fixed-income-recommended-start) |
| Joining a rates desk (curves, swaps, futures) | [Path 2 — Rates Desk](#path-2--rates-desk-curves-and-swaps) |
| In risk / middle office / product control | [Path 3 — Risk Management and Hedging](#path-3--risk-management-and-hedging) |
| Learning credit (CDS, indices, tranches) | [Path 4 — Credit and Structured Credit](#path-4--credit-and-structured-credit) |
| Focused on counterparty risk, XVA, collateral | [Path 5 — Counterparty Risk and XVA](#path-5--counterparty-risk-and-xva) |
| Building quant / modeling depth | [Path 6 — Quant Modeling Depth](#path-6--quant-modeling-depth) |
| Working across FX and rates | [Path 7 — FX and Cross-Currency](#path-7--fx-and-cross-currency) |

Each path below lists prerequisites, a recommended sequence, and what you should be able to do after finishing it.

---

## Path 1 — Core Fixed Income (Recommended Start)

The broadest foundation. If you are new to the subject, read this first; every other path builds on the conventions and risk language introduced here.

**Prerequisites:** basic finance familiarity (present value, bonds as cashflow streams).

**Reading order:**

1. Quoting, conventions, and time value — [Ch. 1](chapters/chapter_01_market_quoting_calendars_cashflow_plumbing.md) → [Ch. 4](chapters/chapter_04_money_market_building_blocks.md)
2. Bond pricing, yields, returns, and spreads — [Ch. 5](chapters/chapter_05_fixed_rate_bond_pricing.md) → [Ch. 8](chapters/chapter_08_spreads_101.md)
3. Repo and Treasury microstructure — [Ch. 9](chapters/chapter_09_repo_funding_engine.md), [Ch. 10](chapters/chapter_10_treasury_microstructure_relative_value.md)
4. First-order risk measures — [Ch. 11](chapters/chapter_11_dv01_pv01_definitions_computation.md) → [Ch. 13](chapters/chapter_13_convexity.md)

**You should be able to:** decode quote formats, build a cashflow schedule with correct settlement and accruals, price a bond from a discount curve, name which spread answers which question, and read a DV01 report critically.

---

## Path 2 — Rates Desk (Curves and Swaps)

For anyone joining or supporting a rates, swaps, or macro desk. Emphasis is on the multi-curve stack and the instruments built on it.

**Prerequisites:** Path 1 or equivalent fluency with discount factors, par rates, and DV01.

**Reading order:**

1. Single-curve bootstrapping and interpolation — [Ch. 17](chapters/chapter_17_curve_construction_bootstrapping_interpolation.md)
2. The modern multi-curve stack — [Ch. 18](chapters/chapter_18_ois_discounting_curve.md) → [Ch. 22](chapters/chapter_22_multi_curve_risk_jacobians.md)
3. Futures as rate instruments — [Ch. 23](chapters/chapter_23_treasury_futures.md), [Ch. 24](chapters/chapter_24_stir_futures_convexity_adjustments.md)
4. Swaps, swap risk, swap-curve RV, and basis trades — [Ch. 25](chapters/chapter_25_interest_rate_swaps_mechanics_valuation.md) → [Ch. 28](chapters/chapter_28_basis_trades.md)

**You should be able to:** build and interrogate a multi-curve stack, value a vanilla swap and explain why discounting and projection curves differ, compute and interpret swap PV01, read swap-spread and asset-swap trades, and navigate STIR convexity adjustments.

---

## Path 3 — Risk Management and Hedging

For risk, product control, and middle office — or anyone who will have to explain P&L to a skeptical manager.

**Prerequisites:** Path 1. Familiarity with swaps ([Ch. 25](chapters/chapter_25_interest_rate_swaps_mechanics_valuation.md)) helps but is not required up front.

**Reading order:**

1. Risk measures from first principles — [Ch. 11](chapters/chapter_11_dv01_pv01_definitions_computation.md) → [Ch. 13](chapters/chapter_13_convexity.md)
2. Curve-shape risk: key rates, hedging, PCA — [Ch. 14](chapters/chapter_14_key_rate_dv01_bucket_exposures.md) → [Ch. 16](chapters/chapter_16_curve_hedging_twists_butterflies_pca.md)
3. Multi-curve deltas and Jacobian mapping — [Ch. 22](chapters/chapter_22_multi_curve_risk_jacobians.md)
4. Swap hedge ratios in practice — [Ch. 26](chapters/chapter_26_swap_pv01_dv01_hedging.md)

**You should be able to:** interrogate any DV01 / PV01 number (what was bumped, by how much, in which units and sign), choose a hedge set that addresses level / slope / curvature, and reason about when a DV01 hedge will quietly fail.

---

## Path 4 — Credit and Structured Credit

From default / recovery fundamentals through CDS mechanics to tranches. This is the longest single-domain track in the book.

**Prerequisites:** Path 1. The discount-factor and PV01 language from Part III is used throughout.

**Reading order:**

1. Credit fundamentals — [Ch. 35](chapters/chapter_35_default_recovery_credit_events.md) → [Ch. 37](chapters/chapter_37_cash_credit_risky_bonds_spreads_cs01.md)
2. CDS mechanics, events, auctions — [Ch. 38](chapters/chapter_38_cds_contract_mechanics.md) → [Ch. 40](chapters/chapter_40_cds_auction_process.md)
3. CDS curves, risk, and single-name RV — [Ch. 41](chapters/chapter_41_cds_indices_mechanics_coupons_rolls.md) → [Ch. 44](chapters/chapter_44_cds_relative_value_trading_frameworks.md)
4. Indices in depth — [Ch. 45](chapters/chapter_45_cds_indices_structure_quoting_lifecycle.md) → [Ch. 47](chapters/chapter_47_hedging_relative_value_cds_indices.md)
5. Tranches and correlation — [Ch. 48](chapters/chapter_48_cdo_tranche_products_product_map.md) → [Ch. 52](chapters/chapter_52_credit_trading_strategies.md)

**You should be able to:** bootstrap a survival curve from spreads, map spread moves into hazard-rate intuition, read a CDS trade confirmation, separate index trading from intrinsic / basis value, and reason about tranche risk in terms of portfolio loss distributions.

---

## Path 5 — Counterparty Risk and XVA

A short, focused track on the valuation adjustments and collateral mechanics that shape every non-cleared derivative trade.

**Prerequisites:** some familiarity with discounting and swaps ([Ch. 17](chapters/chapter_17_curve_construction_bootstrapping_interpolation.md), [Ch. 25](chapters/chapter_25_interest_rate_swaps_mechanics_valuation.md)).

**Reading order:**

1. Exposure, netting, and collateral primitives — [Ch. 32](chapters/chapter_32_counterparty_exposure_basics.md)
2. Collateral discounting and its link to OIS — [Ch. 33](chapters/chapter_33_collateral_discounting_ois.md)
3. XVA taxonomy — [Ch. 34](chapters/chapter_34_xva_overview.md)

**You should be able to:** explain why "fully collateralized" is not the same as "zero exposure," describe MPOR and gap risk, and name what each XVA adjustment is actually pricing.

---

## Path 6 — Quant Modeling Depth

The quant appendices. Take these **after** you are comfortable with curve-based PV and swap valuation — the motivation for every model below is a pricing or hedging question raised earlier in the book.

**Prerequisites:** Paths 1 and 2. Comfort with stochastic calculus helps for A1 and A3.

**Reading order:**

1. No-arbitrage pricing, numeraires, and measure changes — [Appendix A1](chapters/appendix_a1_no_arbitrage_numeraires_measure_changes.md)
2. Short-rate models (Vasicek / CIR / Hull–White) — [Appendix A2](chapters/appendix_a2_short_rate_models.md)
3. HJM framework essentials — [Appendix A3](chapters/appendix_a3_hjm_framework_essentials.md)
4. Market models (LMM, swap market model) — [Appendix A4](chapters/appendix_a4_market_models_lmm_swap_calibration.md)
5. Numerical methods (trees, PDE, Monte Carlo, Fourier) — [Appendix A5](chapters/appendix_a5_numerical_methods_trees_pde_monte_carlo.md)
6. Credit portfolio modeling beyond base correlation — [Appendix A6](chapters/appendix_a6_credit_portfolio_modeling.md)

**You should be able to:** read a term-structure model spec and identify its drift restriction, build calibration intuition for a Hull–White or LMM setup, and choose a numerical scheme appropriate to the payoff.

---

## Path 7 — FX and Cross-Currency

For readers who work across currencies — FX desks, emerging-markets rates, or anyone hedging foreign-currency bond portfolios.

**Prerequisites:** Path 1, plus the cross-currency curves chapter ([Ch. 21](chapters/chapter_21_cross_currency_curves.md)).

**Reading order:**

1. FX spot and forwards via interest differentials — [Ch. 29](chapters/chapter_29_fx_spot_forwards.md)
2. Cross-currency curves as curve constraints — [Ch. 21](chapters/chapter_21_cross_currency_curves.md)
3. FX swaps and cross-currency swaps — [Ch. 30](chapters/chapter_30_fx_swaps_cross_currency_swaps.md)
4. Multi-currency risk aggregation — [Ch. 31](chapters/chapter_31_multi_currency_risk.md)

**You should be able to:** price an FX forward from two curves and a spot, read a cross-currency basis quote, describe how an FX swap funds a foreign-currency position, and aggregate delta across currencies without double-counting.

---

## Combining Paths

The paths are not exclusive. Common combinations:

- **Rates generalist:** Path 1 → Path 2 → Path 3
- **Credit generalist:** Path 1 → Path 4
- **Derivatives quant:** Path 1 → Path 2 → Path 6
- **Risk or middle office with credit coverage:** Path 1 → Path 3 → Path 4
- **Cross-asset desk analyst:** Path 1 → Path 2 → Path 7 → Path 4

When in doubt, read linearly. The chapter order was designed to support it.
