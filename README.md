# Fixed Income: Practice and Theory

A free, desk-oriented book on how fixed income products are **quoted**, **priced**, **risk-managed**, and **traded** — from “99-16+” all the way to curves, swaps, CDS, and tranches.

A trader says a Treasury is “99-16+.” A salesperson quotes a swap rate of “3.25%.” A risk report says “DV01 = $50k.” Those are not just numbers — they’re protocols: conventions that define cashflows, discounting, and what it even *means* to “move rates by 1bp.”

Fixed income often looks like a wall of conventions: day counts, compounding, settlement lags, clean vs dirty prices, par rates, DV01 definitions, curve construction choices. Those conventions are not trivia — they are the interface between a market quote and real cash movements, risk, and P&L.

- Start here: [CONTENTS.md (Table of Contents)](CONTENTS.md)
- Chapters live in: `chapters/`

## The Core Idea

This book treats fixed income as a chain of concrete objects you can actually implement and reason about:

**quote → conventions → schedule → cashflows → curve(s) → PV → risk → hedges → P&L explain**

If you’ve ever seen two systems disagree on DV01, a hedge that “should be flat” still bleed money, or a clean/dirty mix-up create a reconciliation break — it’s usually because one link in this chain was implicit or inconsistent. The chapters aim to make those links explicit, with units, sign conventions, and sanity checks.

## What Makes This Book Different

- **Conventions-first:** rate quotes are treated as “quote objects” with day count, compounding, calendars, and settlement rules (not just a number in a spreadsheet).
- **Curves are the primitive:** discount factors come first; yields and par rates are derived summaries with failure modes.
- **Risk measures are specified:** every DV01/PV01 is tied to a bump object, bump size, units, and sign convention — so risk numbers are comparable and hedgeable.
- **Pitfalls + desk reality:** common breaks are called out explicitly (clean/dirty mixing, forward ≠ forecast, multi-curve “telescoping”, CTD switch traps, “fully collateralized” ≠ zero exposure).
- **References when it matters:** chapters include references to support non-trivial conventions and definitions.

## What You’ll Learn (Concrete Outcomes)

By the end of the core paths, you should be able to:

- Decode common quote formats (32nds, discount-yield bills, swap par rates, CDS spread/upfront conventions) into precise “quote objects”.
- Build schedules and cashflows with explicit calendars, settlement timing, and accrual conventions — and compute invoice amounts.
- Price bonds and swaps using discount factors and curve-based PV (and understand what yield-to-maturity hides).
- Use spread language correctly (G/I/Z/OAS/asset swap) and understand what question each spread is answering.
- Compute and interpret risk measures (DV01/PV01, duration, convexity, key-rate DV01) by specifying **what is being bumped**, by how much, and with what sign.
- Bootstrap and interrogate curves (including OIS discounting, projection curves, tenor basis, cross-currency constraints), and understand why interpolation is a modeling choice with risk consequences.
- Connect trading language to economics: carry/rolldown vs MTM, repo specialness, CTD and implied repo, basis trades in rates and credit.
- Understand counterparty exposure, collateral mechanics, and the intuition behind XVA inputs and failure modes.
- Build credit intuition from default/recovery and hazard rates to CDS mechanics, CDS curves, index basis, and tranche/correlation risk.

## Who This Is For

- New analysts/associates joining rates, credit, or macro desks.
- Middle office (risk, product control, ops) who want the “why” behind breaks and P&L attribution.
- Software engineers building pricing/risk infrastructure who need convention-level correctness.
- Students who know basic finance and want practical fixed income fluency.

Prerequisites vary by chapter. You can start at [Chapter 1](chapters/chapter_01_market_quoting_calendars_cashflow_plumbing.md) with no background; the optional quant appendices assume more math.

## How The Book Is Organized

The full chapter list is in [CONTENTS.md](CONTENTS.md). The book is organized by how practitioners build up the stack:

- Foundations (Ch. 1–4): quoting, calendars, discount factors, zeros/forwards/par rates
- Bonds and rates RV (Ch. 5–10): pricing mechanics, yields, returns, spreads, repo, Treasuries
- Risk and hedging (Ch. 11–16): DV01/duration/convexity, key rates, curve-shape risk
- Curve construction (Ch. 17–22): bootstrapping, OIS discounting, multi-curve, basis, cross-currency, risk mapping
- Futures and swaps (Ch. 23–28): CTD/delivery options, convexity adjustments, swap PV/risk, swap spread RV, basis trades
- FX integration (Ch. 29–31): forwards, FX swaps, cross-currency swaps, multi-currency risk
- Counterparty and XVA (Ch. 32–34): netting/collateral, collateral discounting, CVA/DVA/FVA overview
- Credit and CDS (Ch. 35–47): default/recovery, hazard rates, cash credit, CDS mechanics/curves/RV
- Structured credit (Ch. 48–52): tranches, correlation, risk and strategy framing
- Optional quant appendices (A1–A6): no-arbitrage pricing, term-structure models, numerical methods, credit portfolio modeling

## Quick Index (If You’re Here For One Problem)

| If you want to… | Start here |
|---|---|
| Convert Treasury quotes like `99-16+` into dollars, and understand clean vs dirty settlement cash | [Ch. 1](chapters/chapter_01_market_quoting_calendars_cashflow_plumbing.md), [Ch. 5](chapters/chapter_05_fixed_rate_bond_pricing.md) |
| Understand T-bill “discount yield” quotes (and why they’re not an IRR) | [Ch. 1](chapters/chapter_01_market_quoting_calendars_cashflow_plumbing.md), [Ch. 4](chapters/chapter_04_money_market_building_blocks.md) |
| Understand the zero/forward/par “triangle” and why forwards aren’t forecasts | [Ch. 3](chapters/chapter_03_zero_forward_par_rates_triangle.md) |
| Understand “DV01/PV01” in a risk report (and why two DV01s can disagree) | [Ch. 11](chapters/chapter_11_dv01_pv01_definitions_computation.md) |
| Decompose bond returns into carry, rolldown, curve moves, and spread changes | [Ch. 7](chapters/chapter_07_bond_return_decomposition.md) |
| Speak spread correctly (G/I/Z/OAS/ASW) and know which spread answers which question | [Ch. 8](chapters/chapter_08_spreads_101.md) |
| Understand repo, specialness, and how funding shows up in relative value | [Ch. 9](chapters/chapter_09_repo_funding_engine.md) |
| Understand Treasury futures: CTD, conversion factors, invoice pricing, implied repo, CTD switch risk | [Ch. 23](chapters/chapter_23_treasury_futures.md) |
| Build curves (bootstrapping + interpolation) and understand why curve choices affect valuation and hedges | [Ch. 17](chapters/chapter_17_curve_construction_bootstrapping_interpolation.md) |
| Work in a modern multi-curve setup (OIS discounting, projection curves, tenor basis, cross-currency) | [Ch. 18](chapters/chapter_18_ois_discounting_curve.md)–[Ch. 22](chapters/chapter_22_multi_curve_risk_jacobians.md) |
| Avoid the classic multi-curve swap PV mistake (“telescoping” a float leg that uses a different curve) | [Ch. 25](chapters/chapter_25_interest_rate_swaps_mechanics_valuation.md) |
| Understand counterparty exposure and why “daily VM” still leaves residual risk (MPOR/gap risk) | [Ch. 32](chapters/chapter_32_counterparty_exposure_basics.md), [Ch. 34](chapters/chapter_34_xva_overview.md) |
| Learn CDS mechanics (premium accrual, settlement, auctions, upfront-plus-coupon quoting) | [Ch. 38](chapters/chapter_38_cds_contract_mechanics.md)–[Ch. 40](chapters/chapter_40_cds_auction_process.md) |
| Bootstrap a CDS survival curve and connect spreads to hazard rates/survival | [Ch. 36](chapters/chapter_36_survival_probabilities_hazard_rates.md), [Ch. 42](chapters/chapter_42_bootstrapping_cds_survival_curve.md) |
| Get structured credit intuition (attachment/detachment, tranche PV, correlation risk) | [Ch. 48](chapters/chapter_48_cdo_tranche_products_product_map.md)–[Ch. 51](chapters/chapter_51_tranche_risk.md) |

## Learning Paths

Pick a path based on your goal. These are shortcuts; the canonical chapter ordering is [CONTENTS.md](CONTENTS.md).

### Path 1: Core Fixed Income (Recommended Start)

- Start with quoting/conventions and time value: [Ch. 1](chapters/chapter_01_market_quoting_calendars_cashflow_plumbing.md)–[Ch. 4](chapters/chapter_04_money_market_building_blocks.md)
- Learn bond mechanics and language: [Ch. 5](chapters/chapter_05_fixed_rate_bond_pricing.md)–[Ch. 8](chapters/chapter_08_spreads_101.md)
- Learn risk measures that don’t lie to you: [Ch. 11](chapters/chapter_11_dv01_pv01_definitions_computation.md)–[Ch. 13](chapters/chapter_13_convexity.md)

### Path 2: Rates Desk / Curve Construction + Swaps

- Single-curve mechanics and interpolation: [Ch. 17](chapters/chapter_17_curve_construction_bootstrapping_interpolation.md)
- Multi-curve stack (OIS, projection, basis, cross-currency, risk mapping): [Ch. 18](chapters/chapter_18_ois_discounting_curve.md)–[Ch. 22](chapters/chapter_22_multi_curve_risk_jacobians.md)
- Swaps, swap risk, and basis framing: [Ch. 25](chapters/chapter_25_interest_rate_swaps_mechanics_valuation.md)–[Ch. 28](chapters/chapter_28_basis_trades.md)
- Futures as rate instruments (optional but practical): [Ch. 23](chapters/chapter_23_treasury_futures.md), [Ch. 24](chapters/chapter_24_stir_futures_convexity_adjustments.md)

### Path 3: Risk Management and Hedging (Desk-Ready)

- Core risk measures and curve-shape intuition: [Ch. 11](chapters/chapter_11_dv01_pv01_definitions_computation.md)–[Ch. 16](chapters/chapter_16_curve_hedging_twists_butterflies_pca.md)
- Multi-curve deltas and Jacobian mapping: [Ch. 22](chapters/chapter_22_multi_curve_risk_jacobians.md)
- Swap PV01/DV01 and hedge ratios: [Ch. 26](chapters/chapter_26_swap_pv01_dv01_hedging.md)

### Path 4: Credit + CDS → Structured Credit

- Credit foundations: [Ch. 35](chapters/chapter_35_default_recovery_credit_events.md)–[Ch. 37](chapters/chapter_37_cash_credit_risky_bonds_spreads_cs01.md)
- CDS mechanics → curves → RV: [Ch. 38](chapters/chapter_38_cds_contract_mechanics.md)–[Ch. 47](chapters/chapter_47_hedging_relative_value_cds_indices.md)
- Tranches/correlation/risk: [Ch. 48](chapters/chapter_48_cdo_tranche_products_product_map.md)–[Ch. 52](chapters/chapter_52_credit_trading_strategies.md)

### Path 5: Counterparty + XVA (Practical Intuition)

- Exposure and collateral primitives: [Ch. 32](chapters/chapter_32_counterparty_exposure_basics.md)
- Collateral discounting and why it matters: [Ch. 33](chapters/chapter_33_collateral_discounting_ois.md)
- XVA taxonomy and pitfalls: [Ch. 34](chapters/chapter_34_xva_overview.md)

### Path 6: Quant Appendices (Optional Depth)

- Take after you’re comfortable with curve-based PV and swaps.
- Start with: [Appendix A1](chapters/appendix_a1_no_arbitrage_numeraires_measure_changes.md)

### Path 7: FX + Cross-Currency (Rates × FX)

- FX forwards and interest differentials: [Ch. 29](chapters/chapter_29_fx_spot_forwards.md)
- Cross-currency curves (CIP, FX forwards, basis as constraints): [Ch. 21](chapters/chapter_21_cross_currency_curves.md)
- FX swaps and cross-currency swaps: [Ch. 30](chapters/chapter_30_fx_swaps_cross_currency_swaps.md)
- Multi-currency risk aggregation: [Ch. 31](chapters/chapter_31_multi_currency_risk.md)

## How To Use This Repo

- Read in GitHub: start with [CONTENTS.md](CONTENTS.md) and follow the chapter links.
- Chapters are written for self-study: introductions, learning objectives, worked examples, common pitfalls, desk-reality callouts, and references.
- Many chapters include prerequisites and follow-on links to help you skip around without getting lost.

## Contributions / Errata

If you find a mistake (especially around conventions, units, and sign), please open an issue or PR. High-value contributions include:

- Corrections and clarifications (ideally with a source when the convention is non-trivial or market-dependent).
- Worked examples and sanity checks that make a tricky concept “stick”.
- Fixing broken links, typos, or unclear diagrams-in-words.

## Disclaimer

This book is for education. It is not investment advice.

Market conventions differ across products, jurisdictions, venues, and time. Always confirm details against authoritative documentation (issuer terms, ISDA/ARRC, CCP rulebooks, exchange rulebooks, and official market publications) before relying on an implementation in production.
