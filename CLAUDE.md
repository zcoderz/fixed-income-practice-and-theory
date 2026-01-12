# Fixed Income: Practice and Theory — Agent Instructions

This file guides AI agents in constructing chapter content for a comprehensive fixed income textbook/notes series.

---

## Project Overview

**Goal:** Create a practitioner-oriented fixed income textbook that bridges theory and desk practice, covering bonds, swaps, curves, credit, and structured products.

**Target Audience:**
- Quantitative analysts entering rates/credit desks
- Graduate students in financial engineering
- Practitioners seeking rigorous yet practical foundations

**Structure:** See `CONTENTS.md` for the full 52-chapter outline across 10 parts plus 6 quant appendices.

---

## Reference Books — Source Map

All source materials are in the `books/` directory. Here is the mapping of books to chapter topics:

### Primary Sources (Fixed Income Core)

| Book | Path | Primary Use |
|------|------|-------------|
| **Tuckman** — *Fixed Income Securities* | `books/05_advanced/fixed_income/fixed_income_securities_tuckman.md` | **Parts I-III, V**: Bond pricing, yields, DV01, duration, convexity, repo, Treasury market, futures |
| **Hull** — *Options, Futures, and Other Derivatives* | `books/02_portfolio_risk/portfolio_risk/options_futures_and_other_derivatives.md` | **Parts I-IV, VII**: Interest rates, day counts, compounding, swaps, forwards, credit risk, XVAs |
| **Andersen & Piterbarg** — *Interest Rate Modeling* (3 vols) | `books/05_advanced/fixed_income/interest_rate_modeling.md` | **Parts IV, V, Appendices**: Curve construction, multi-curve, HJM, LMM, numerical methods |
| **O'Kane** — *Modeling Single-Name and Multi-Name Credit Derivatives* | `books/05_advanced/fixed_income/modeling_single_name_and_multi_name_credit_derivatives.md` | **Parts VIII-X**: CDS mechanics, pricing, survival curves, indices, tranches, CDOs |
| **Brigo & Mercurio** — *Interest Rate Models: Theory and Practice* | `books/05_advanced/fixed_income/interest_rate_models_theory_and_practice.md` | **Parts IV, V, Appendices**: Short-rate models, HJM, market models, calibration |

### Secondary Sources

| Book | Path | Use For |
|------|------|---------|
| **Veronesi** — *Modeling Fixed Income Securities* | `books/05_advanced/fixed_income/modeling_fixed_income_securities_and_interest_rate_options.md` | Bond options, term structure modeling |
| **McNeil et al** — *Quantitative Risk Management* | `books/02_portfolio_risk/portfolio_risk/quantitative_risk_management.md` | Credit risk fundamentals, portfolio credit |
| **Cochrane** — *Asset Pricing* | `books/03_pricing_core/computational_finance/asset_pricing_cochrane.md` | No-arbitrage theory, stochastic discount factors |
| **Glasserman** — *Monte Carlo Methods in Financial Engineering* | `books/03_pricing_core/computational_finance/monte_carlo_methods_financial_engineering.md` | Simulation, variance reduction |
| **Hirsa** — *Computational Methods in Finance* | `books/03_pricing_core/computational_finance/computational_methods_in_finance_hirsa.md` | Numerical methods, FFT pricing |

---

## Chapter-to-Book Mapping

### Part I — Foundations (Chapters 1-4)

| Chapter | Primary Sources | Key Sections to Search |
|---------|-----------------|------------------------|
| Ch 1: Market Quoting, Calendars | Tuckman Ch 1-4, Hull Ch 4, Andersen Vol 1 Ch 4-5 | "accrued interest", "day count", "settlement", "clean price", "dirty price" |
| Ch 2: Time Value, Discount Factors | Tuckman Ch 1, Hull Ch 4 | "discount factor", "present value", "law of one price", "replication" |
| Ch 3: Zero/Forward/Par Rates | Tuckman Ch 2, Hull Ch 4, Andersen Vol 1 | "spot rate", "forward rate", "par rate", "bootstrap" |
| Ch 4: Compounding, Day Count | Hull Ch 4, Tuckman Ch 1 | "compounding", "continuous", "semiannual", "ACT/360", "30/360" |

### Part II — Bonds (Chapters 5-10)

| Chapter | Primary Sources | Key Sections to Search |
|---------|-----------------|------------------------|
| Ch 5: Bond Pricing | Tuckman Ch 1-4, Hull Ch 4 | "bond price", "cashflow", "PV", "invoice price" |
| Ch 6: YTM and Yield Risk | Tuckman Ch 3-5 | "yield to maturity", "IRR", "yield-based DV01" |
| Ch 7: Carry, Rolldown | Tuckman Ch 3, 8 | "carry", "rolldown", "return decomposition" |
| Ch 8: Spread Measures | Tuckman Ch 4, 17-18, O'Kane Ch 4-5 | "G-spread", "I-spread", "Z-spread", "OAS", "asset swap" |
| Ch 9: Repo | Tuckman Ch 15-16 | "repo", "general collateral", "specials", "fails" |
| Ch 10: Treasury Microstructure | Tuckman Ch 15-16 | "on-the-run", "off-the-run", "auction", "liquidity" |

### Part III — Risk Measures (Chapters 11-16)

| Chapter | Primary Sources | Key Sections to Search |
|---------|-----------------|------------------------|
| Ch 11: DV01/PV01 | Tuckman Ch 5-6, Hull Ch 4 | "DV01", "PV01", "price sensitivity", "basis point" |
| Ch 12: Duration | Tuckman Ch 5, Hull Ch 4 | "Macaulay duration", "modified duration" |
| Ch 13: Convexity | Tuckman Ch 5-6 | "convexity", "second derivative", "gamma" |
| Ch 14: Key-Rate DV01 | Tuckman Ch 6, Andersen Vol 3 | "key rate", "bucket", "partial DV01" |
| Ch 15: DV01 Hedging | Tuckman Ch 6-7 | "hedge ratio", "duration matching" |
| Ch 16: Curve Hedging | Tuckman Ch 6-7, Andersen Vol 3 | "butterfly", "PCA", "level slope curvature" |

### Part IV — Curve Construction (Chapters 17-22)

| Chapter | Primary Sources | Key Sections to Search |
|---------|-----------------|------------------------|
| Ch 17: Bootstrapping | Hull Ch 4, Andersen Vol 1 Ch 4-6 | "bootstrap", "interpolation", "spline" |
| Ch 18: OIS Curve | Andersen Vol 1 Ch 6, Hull Ch 4 | "OIS", "overnight", "discounting" |
| Ch 19: Projection Curves | Andersen Vol 1 Ch 6 | "projection", "LIBOR", "SOFR", "multi-curve" |
| Ch 20: Tenor Basis | Andersen Vol 1 Ch 6 | "basis swap", "tenor basis", "1M vs 3M" |
| Ch 21: Cross-Currency | Andersen Vol 1, Hull Ch 7 | "cross-currency", "CIP", "FX forward", "xccy basis" |
| Ch 22: Multi-Curve Risk | Andersen Vol 1 Ch 6, Vol 3 | "Jacobian", "par-point delta" |

### Part V — Futures and Swaps (Chapters 23-28)

| Chapter | Primary Sources | Key Sections to Search |
|---------|-----------------|------------------------|
| Ch 23: Treasury Futures | Tuckman Ch 19-20 | "CTD", "conversion factor", "delivery option" |
| Ch 24: STIR Futures | Tuckman Ch 17, Hull Ch 6 | "Eurodollar", "convexity adjustment", "futures vs forward" |
| Ch 25: IRS Mechanics | Hull Ch 7, Andersen Vol 1 | "swap", "fixed leg", "floating leg" |
| Ch 26: Swap PV01 | Tuckman Ch 18, Andersen Vol 1 | "swap DV01", "annuity" |
| Ch 27: Swap Spreads | Tuckman Ch 18 | "swap spread", "asset swap spread" |
| Ch 28: Basis Trades | Tuckman Ch 18, Andersen Vol 1 | "OIS-IBOR basis", "swap spread trade" |

### Part VI — FX and Cross-Currency (Chapters 29-31)

| Chapter | Primary Sources | Key Sections to Search |
|---------|-----------------|------------------------|
| Ch 29: FX Forwards | Hull Ch 5, Andersen Vol 1 | "FX forward", "forward points", "CIP" |
| Ch 30: Cross-Currency Swaps | Andersen Vol 1, Hull Ch 7 | "cross-currency swap", "xccy", "basis" |
| Ch 31: Multi-Currency Risk | Andersen Vol 3 | "FX delta", "rates delta", "cross-gamma" |

### Part VII — Counterparty Risk (Chapters 32-34)

| Chapter | Primary Sources | Key Sections to Search |
|---------|-----------------|------------------------|
| Ch 32: Counterparty Exposure | Hull Ch 9, 24 | "PFE", "netting", "collateral", "margin" |
| Ch 33: Collateral Discounting | Hull Ch 9, Andersen Vol 1 Ch 6 | "CSA", "OIS discounting", "collateralized" |
| Ch 34: XVA Overview | Hull Ch 9 | "CVA", "DVA", "FVA", "XVA" |

### Part VIII — Credit Fundamentals (Chapters 35-37)

| Chapter | Primary Sources | Key Sections to Search |
|---------|-----------------|------------------------|
| Ch 35: Default and Recovery | O'Kane Ch 3, Hull Ch 24 | "default", "recovery", "credit event" |
| Ch 36: Survival Probabilities | O'Kane Ch 3, 6-7 | "survival", "hazard rate", "intensity" |
| Ch 37: Cash Credit | O'Kane Ch 4, Hull Ch 24 | "risky bond", "credit spread", "spread duration" |

### Part IX — CDS (Chapters 38-44)

| Chapter | Primary Sources | Key Sections to Search |
|---------|-----------------|------------------------|
| Ch 38: CDS Mechanics | O'Kane Ch 5, Hull Ch 25 | "premium leg", "protection leg", "accrued" |
| Ch 39: Credit Events | O'Kane Ch 5, Hull Ch 25 | "restructuring", "bankruptcy", "failure to pay" |
| Ch 40: CDS Auction | O'Kane Ch 5 | "auction", "recovery determination" |
| Ch 41: CDS Pricing | O'Kane Ch 6, Hull Ch 25 | "par spread", "upfront", "risky PV01" |
| Ch 42: Survival Curve Bootstrap | O'Kane Ch 7 | "bootstrap", "hazard rate", "piecewise" |
| Ch 43: CDS Risks | O'Kane Ch 8 | "CS01", "jump-to-default", "recovery risk" |
| Ch 44: CDS Relative Value | O'Kane Ch 8 | "curve trade", "steepener", "flattener" |

### Part X — Indices and Tranches (Chapters 45-52)

| Chapter | Primary Sources | Key Sections to Search |
|---------|-----------------|------------------------|
| Ch 45: CDS Indices | O'Kane Ch 9-10 | "CDX", "iTraxx", "index roll" |
| Ch 46: Intrinsic Spread | O'Kane Ch 10 | "intrinsic", "index basis" |
| Ch 47: Index Hedging | O'Kane Ch 10 | "single-name hedge", "index hedge" |
| Ch 48: CDO/Tranches | O'Kane Ch 11-12 | "tranche", "attachment", "detachment", "waterfall" |
| Ch 49: Tranche Concepts | O'Kane Ch 12-13 | "expected loss", "tranche PV" |
| Ch 50: Correlation | O'Kane Ch 13-14 | "base correlation", "Gaussian copula" |
| Ch 51: Tranche Risk | O'Kane Ch 14-15 | "tranche delta", "correlation risk" |
| Ch 52: Credit Strategies | O'Kane Ch 8, 10, 15 | "basis trade", "curve trade", "correlation trade" |

### Quant Appendices (A1-A6)

| Appendix | Primary Sources | Key Sections to Search |
|----------|-----------------|------------------------|
| A1: No-Arbitrage | Andersen Vol 1 Ch 1, Brigo Ch 1-2 | "martingale", "numeraire", "measure change" |
| A2: Short-Rate Models | Brigo Ch 3-4, Andersen Vol 2 | "Vasicek", "CIR", "Hull-White" |
| A3: HJM | Brigo Ch 5, Andersen Vol 2 | "HJM", "drift restriction", "volatility" |
| A4: Market Models | Brigo Ch 6-7, Andersen Vol 2 | "LMM", "swap market model", "calibration" |
| A5: Numerical Methods | Andersen Vol 1 Ch 2-3, Glasserman | "tree", "PDE", "Monte Carlo" |
| A6: Credit Portfolio | O'Kane Ch 13-15, McNeil | "factor model", "copula", "dynamic intensity" |

---

## Style and Tone Guidelines

### Voice
- **Practitioner-first**: Write as if explaining to a new desk analyst
- **Direct and concise**: No fluff, no excessive hedging
- **Precise but accessible**: Use correct terminology, but always define it first
- **Honest about uncertainty**: Label speculation clearly; admit when conventions vary

### Structure (Required for Each Chapter)

Every chapter MUST include these sections in order:

```markdown
# Chapter N: Title

---

## Fact Classification
### (A) Verified Facts (Source-Backed)
### (B) Reasoned Inference (Derived from A)
### (C) Speculation (Clearly Labeled; Minimal)

---

## Conventions & Notation
- Notation glossary table
- Defaults used in examples

---

## Core Concepts
### 1) Concept Name
**Formal Definition:** ...
**Intuition:** ...
**Trading / Risk / Portfolio Practice:** ...

[Repeat for each core concept]

---

## Math and Derivations
- Step-by-step derivations with boxed key formulas
- Unit checks and sanity checks

---

## Worked Examples
- At least 3-5 numeric examples
- Show all intermediate steps
- Include edge cases (e.g., negative rates)

---

## Practical Notes
- Common quoting gotchas
- Implementation pitfalls
- Verification tests / sanity checks

---

## Summary & Recall
### 10-Bullet Executive Summary
### Cheat Sheet of Formulas
### Flashcards (15-20 Q/A pairs)

---

## Mini Problem Set
- 8-12 questions, increasing difficulty
- Solution sketches for first few

---

## Source Map
### (A) Verified Facts — cite specific sources
### (B) Reasoned Inference — note derivation logic
### (C) Speculation — flag uncertainties
```

### Formatting Rules

1. **Math**: Use LaTeX notation with `$...$` for inline and `$$...$$` for display
2. **Key formulas**: Box important results with `$$\boxed{...}$$`
3. **Tables**: Use markdown tables for notation, examples, comparisons
4. **Code**: If including code snippets, use Python with clear comments
5. **Cross-references**: Reference other chapters as "Ch. N" or "see Chapter N"

### Content Principles

1. **Always verify against sources**: Before writing a "fact," search the relevant book(s)
2. **Show the derivation**: Don't just state formulas—derive them step by step
3. **Include sanity checks**: Unit checks, boundary cases, sign checks
4. **Connect to practice**: Every concept should link to how it's used on a desk
5. **Acknowledge convention variation**: Markets differ; flag when conventions vary

---

## Agent Instructions

### When Constructing a New Chapter

1. **Read the chapter outline** in `CONTENTS.md` for scope and topics
2. **Search reference books** using the mapping above:
   - Use `Grep` to search for key terms
   - Use `Read` to examine relevant sections
3. **Cross-reference existing chapters** (1-4) for style consistency
4. **Follow the structure template** exactly
5. **Cite sources** in the Source Map section

### Search Strategy for Books

When searching for content in `books/`:

```bash
# Search for a concept across all fixed income books
Grep "discount factor" books/05_advanced/fixed_income/

# Search Hull specifically
Grep "day count" books/02_portfolio_risk/portfolio_risk/options_futures_and_other_derivatives.md

# Search O'Kane for CDS content
Grep "premium leg" books/05_advanced/fixed_income/modeling_single_name_and_multi_name_credit_derivatives.md
```

### Quality Checks Before Completing a Chapter

- [ ] All core concepts have Formal Definition + Intuition + Practice sections
- [ ] All key formulas are boxed and have unit checks
- [ ] At least 3 worked examples with full numeric detail
- [ ] Flashcards cover all key definitions and formulas
- [ ] Problem set has 8-12 questions
- [ ] Source map attributes all facts to specific sources
- [ ] Notation is consistent with Chapters 1-4

---

## File Structure

```
fixed_income_practice_and_theory/
├── CLAUDE.md              # This file — agent instructions
├── CONTENTS.md            # Full book outline
├── .gitignore             # Excludes books/
├── chapters/
│   ├── chapter_01_*.md
│   ├── chapter_02_*.md
│   ├── chapter_03_*.md
│   ├── chapter_04_*.md
│   └── ...                # Future chapters
└── books/                 # Reference materials (not in git)
    └── 05_advanced/fixed_income/
        ├── fixed_income_securities_tuckman.md
        ├── interest_rate_modeling.md
        ├── interest_rate_models_theory_and_practice.md
        ├── modeling_single_name_and_multi_name_credit_derivatives.md
        └── ...
```

---

## Key Conventions to Maintain Across All Chapters

### Notation Consistency

| Symbol | Meaning | First Defined |
|--------|---------|---------------|
| $P(t,T)$ or $d(T)$ | Discount factor | Ch 2 |
| $z_c(T)$ | Continuous zero rate | Ch 3 |
| $F_s(0;T_1,T_2)$ | Simple forward rate | Ch 3 |
| $P$ (price context) | Clean/quoted price | Ch 1 |
| $P_{\text{full}}$ or $P^{\text{dirty}}$ | Dirty/invoice price | Ch 1, 4 |
| $AI$ | Accrued interest | Ch 1, 4 |
| $\tau$ or $\alpha$ | Year fraction / accrual factor | Ch 1, 4 |
| $c$ | Coupon rate (annual) | Ch 1 |
| $m$ | Compounding frequency | Ch 3, 4 |

### Terminology Preferences

| Prefer | Avoid |
|--------|-------|
| "discount factor" | "discount rate" (ambiguous) |
| "clean price" or "quoted price" | "flat price" (less common) |
| "dirty price" or "invoice price" | "full price" (ambiguous) |
| "continuously compounded" | "continuous" (specify) |
| "semiannual compounding" | "semi-annual" (hyphenation varies) |

---

## Example: How to Approach a New Chapter

If asked to write **Chapter 5: Fixed-Rate Bond Pricing**:

1. **Check CONTENTS.md** for the chapter outline:
   - Cashflow schedule → PV with a curve
   - Accrued interest mechanics
   - Price quotation conventions

2. **Search sources**:
   ```
   Grep "bond price" books/05_advanced/fixed_income/fixed_income_securities_tuckman.md
   Grep "cashflow" books/02_portfolio_risk/portfolio_risk/options_futures_and_other_derivatives.md
   ```

3. **Read relevant sections** in Tuckman Ch 1-4, Hull Ch 4

4. **Cross-reference Chapters 1-4** for:
   - How we defined discount factors (Ch 2)
   - How we handle accrued interest (Ch 1, 4)
   - Notation for prices ($P$, $P_{\text{full}}$, $AI$)

5. **Draft the chapter** following the template

6. **Verify** all formulas with source citations

---

*Last Updated: January 2026*
