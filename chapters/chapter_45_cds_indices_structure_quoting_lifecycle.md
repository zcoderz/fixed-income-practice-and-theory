# Chapter 45: CDS Indices — Structure, Quoting, and Lifecycle (On-the-Run vs Off-the-Run, Coupon vs Upfront, Rolls)

---

## Conventions & Notation

### Notation Glossary

| Symbol | Definition |
|--------|------------|
| $N$ | Index trade notional (USD) |
| $M$ | Number of index constituents |
| $C$ or $c$ | Annualized running coupon rate in decimal (e.g., 100 bp $= 0.01$) |
| bp | Basis point $= 10^{-4}$ |
| $\alpha$ | Accrual fraction (year fraction for a coupon period under Actual/360) |
| $U\%$ | Upfront quote as percent of notional exchanged at settlement |
| $U_\$$ | Dollar upfront $= N \times U\%$ |
| $t$ | Trade date |
| $\tau$ | Default time |
| $FP$ | Auction "final price" (value per 100 of deliverable) used for cash settlement |

### Defaults Used in Examples

| Convention | Default |
|------------|---------|
| Premium payments | Quarterly with Actual/360 accrual |
| Notional currency | USD (examples use $\$mm$) |
| Coupon notation | Annualized decimal |

---

## Fact Classification

### (A) Verified Facts (Source-Backed)

- Index series are issued/rolled every six months, with on-the-run liquidity concentrating in the most recent series
- A "$T$-year" on-the-run index does not have exactly $T$ years remaining at issue; at issuance it is typically $T+3$ months and becomes $T-3$ months by the next roll
- The index coupon is fixed, set close to fair value at inception and often chosen as a multiple of 5 bp
- Premium is usually paid quarterly on Actual/360
- Defaults reduce outstanding notional by $1/M$, and the contract includes an accrued-premium-at-default component
- For investment-grade index contracts, the market may quote an "index spread" (a flat spread concept) and infer an upfront; for lower credit quality indices a "bond price" style quote (price minus 100) can be used to avoid disputes about index PV01 used to translate spread to upfront
- Cash settlement via auction: a two-stage auction can determine the mid-market value of a deliverable bond after a credit event; cash payoff equals par minus that value (e.g., 35 per 100 $\Rightarrow$ 65% payout)

### (B) Reasoned Inference (Derived from A)

- "On-the-run" can be treated as the most recently issued series; "off-the-run" as previous series (still tradable, typically less liquid), consistent with the roll description and liquidity drop after roll
- With fixed coupons, the market's price discovery naturally shifts into the upfront (or "price") dimension, because coupon is no longer the free variable

### (C) Speculation (Clearly Labeled; Minimal)

- I'm not sure about the current standardized coupons, exact roll dates by calendar day for every index family, exact membership rules, or settlement day conventions for your specific trade. To be certain, we would need the relevant index rulebook (e.g., Markit index documentation) and the applicable ISDA Definitions and (if relevant) auction protocols.

---

## Setup

**Conventions used in this chapter:**

- **Mechanics-first:** we focus on contract structure, quoting, lifecycle, and default handling, not full calibration
- **"Long credit" vs "long protection":**
  - "Long credit" (credit risk exposure) corresponds economically to selling protection (receiving premium, paying losses on default)
  - "Long protection" corresponds to buying protection (paying premium, receiving default payoff)
- Where the sources' "buyer/seller" language is ambiguous, we will explicitly restate which side pays premium and which side pays loss

---

## Core Concepts

### 1) CDS Index

**Formal Definition:**

A standardized multi-name credit derivative referencing a portfolio of $M$ reference entities, structured so premium and default-loss cashflows are aggregated and standardized. The book models protection losses as an equal-weighted average across names, i.e. default of name $m$ generates a loss contribution scaled by $1/M$.

**Intuition:**

"One trade to express a view on broad credit." Instead of trading many single-name CDS, you trade one instrument representing a basket.

**Trading/Risk/Portfolio Practice:**

Traders use indices to take macro credit views and to hedge broader credit exposures; liquidity can make the index a "lead" instrument relative to single names.

---

### 2) Series / Version / On-the-Run vs Off-the-Run

**Formal Definition:**

| Term | Definition |
|------|------------|
| **Series (issue)** | A particular issuance of the index with a fixed constituent list for its lifetime (except default removals) |
| **On-the-run** | The most recently issued series (inferred from the "roll into the new on-the-run" description) |
| **Off-the-run** | Older series after the next roll |

**Intuition:**

Like benchmark bonds, the newest contract concentrates dealer inventory, hedging flows, and quoted liquidity.

**Trading/Risk/Portfolio Practice:**

Hedging an off-the-run index with the on-the-run index can create mismatch because series constituents can differ.

---

### 3) Fixed Coupon vs "Index Spread"

**Formal Definition:**

| Term | Definition |
|------|------------|
| **Contractual coupon $C$** | Fixed premium rate specified by the index contract, paid on outstanding notional |
| **Index spread quote** | A market quote representing a "flat spread" level at which the index would price like a single CDS; used especially for investment-grade indices |

**Intuition:**

Coupon is standardized (set in 5 bp increments near inception fair value), so price variation is carried by upfront (or by a quoted spread translated into upfront).

**Trading/Risk/Portfolio Practice:**

A spread quote usually requires an index PV01 (annuity) to translate into upfront; bond-price quoting avoids PV01 disagreements.

---

### 4) Roll

**Formal Definition:**

Replacing the on-the-run series with a newly issued series (typically every six months), and market participants often "roll" positions by selling the old series and buying the new.

**Intuition:**

Keeps index representative and liquid; removes defaulted/illiquid names over time.

**Trading/Risk/Portfolio Practice:**

Rolling can cause P&L due to composition change and maturity extension (new series is ~6 months longer).

---

## What a CDS Index Is (Structure)

### Index as a Portfolio of Reference Entities

The index references $M$ names; the book's index-leg expressions and mechanics treat each name as contributing $1/M$ of the portfolio notional and loss.

**Notional allocation/weighting:** Equal weighting is the convention used in the book's mechanics (loss and notional reductions by $1/M$).

> **Note:** If your market index uses different weights: I'm not sure. We would need the index rulebook to confirm.

### Series, Versioning, and "On-the-Run" Terminology

**What it means:**

- Each issuance defines a constituent set for the index's life, except default removals
- On-the-run refers to the current, most recently issued series (inferred from roll mechanics)

**Why liquidity concentrates in the on-the-run series:**

- Market participants typically roll into the new on-the-run series, concentrating trading activity there
- In index options, expiries beyond six months are rare because liquidity "drops immediately after the next roll," raising hedging costs—evidence of on-the-run dominance

**What "off-the-run" implies for liquidity/hedging:**

- Off-the-run positions may be hedged with on-the-run, but constituent differences can create mismatch risk

---

## Lifecycle Overview

### Launch of a New Series (Roll)

- Indices roll every six months; investors roll by selling old and buying new
- Issuance timing example in the book: 5Y index swaps issued in March and September and mature in June and December, respectively (five years out)
- Hull also describes standard index portfolio updates on March 20 and September 20 for two major portfolios (CDX NA IG and iTraxx Europe)

### Trading During the Series' Life

- At trade date $t$, the contract is described as cash-settled three days later; upfront exchanges hands at settlement and coupon flows run thereafter

### What Happens When Constituents Default (Conceptual)

When a name defaults, the mechanics in the book show:

1. A default-related exchange (described in the figure narrative) and an auction-based cash fallback if deliverables are scarce
2. Coupon accrued on the defaulted name is paid
3. Outstanding notional (and hence future coupon amounts) is reduced by $1/M$

### Maturity/End of Life

- Premium payments continue until maturity $T$ if no credit events; otherwise premium payments reduce as notional is reduced

### Clarify What Is Standardized vs What Can Vary (Only as Sourced)

**Standardized in the book's description:**

- Fixed coupon chosen near inception fair value and in multiples of 5 bp
- Quarterly premium on Actual/360
- Rolling every six months

**Can vary across index families/regions:**

- Credit-event definitions can differ (e.g., for one US index, restructuring excluded in the index contract though it may be included in single-name standards), creating basis

**I'm not sure about:**

- Exact membership rules, exact standard recovery assumptions, and detailed settlement calendars for your particular index—needs the index rulebook + ISDA definitions

---

## Quoting: Coupon vs Upfront (How Indices Trade)

### Distinguish Clearly

#### Running Coupon $C$

The contract pays a fixed coupon (contractual index spread) $C(T)$, paid on outstanding notional, with frequency and accrual per index specification.

In the mechanics description, coupon is typically paid quarterly on Actual/360.

> **Standardized coupon levels (e.g., "always 100 bp"):** I'm not sure. The book states coupons are set near fair value and in 5 bp increments, but does not assert a universal coupon set across all indices and time.

#### Upfront Payment $U\%$

At settlement, an upfront payment equals the contract's value (can be negative so that one party receives upfront).

#### Quoted "Spread" Language vs Coupon+Upfront Quoting

- **Investment grade indices:** market may quote an "index spread" (flat spread concept) used to translate into upfront relative to the fixed coupon
- **Some lower quality indices:** may use a bond-price convention where "upfront" is computed as price minus 100 (per 100 notional), avoiding PV01 disputes
- Hull notes that CDS and CDS indices "trade like bonds" with a fixed coupon specified for standard transactions

### Explain Economically

**Why standardizing the coupon can shift price discovery to upfront:**

If $C$ is fixed by contract design, ensuring the contract can trade across dealers and platforms, then market clearing must occur via upfront (or equivalently via a quoted spread translated into upfront). This is exactly the reason "bond-like" quoting conventions exist for CDS/CDS indices in practice.

**How upfront converts to a dollar amount:**

$$\boxed{U_\$ = N \times U\%}$$

(with $U\%$ in fractional units, e.g. 3.25% $= 0.0325$)

### Accrued Premium Mechanics (Between Coupon Dates)

**Supported mechanics:**

- **Clean vs full MTM convention:** clean MTM excludes accrued; full MTM includes it
- The sign of accrued depends on whether you are long or short protection (single-name CDS explanation, but used here as a generic quoting convention)

**Generic accrual math** (used if you don't have the exact schedule/day count details):

If the annual coupon is $c$ (decimal), and the elapsed accrual fraction since last coupon date is $\Delta$, then accrued premium magnitude is:

$$\text{Accrued magnitude} = N \times c \times \Delta$$

For a long protection position, O'Kane's sign convention for quoting accrued is negative (you owe accrued when transferring the position).

**Default-time accrual** (cashflow-relevant, not just quoting):

The contract may require payment of coupon accrued up to default time (a contingent cashflow) and this is distinct from "clean price" quoting conventions.

---

## Rolls and Index Lifecycle in Practice

### What "Roll" Means and Why It Happens

**Refreshing constituents / removing defaulted names:**

- Within a given issuance, constituents stay fixed except defaulted names are removed without replacement
- Across rolls, constituents can change: names may be removed for criteria like downgrade out of IG, a credit event, or reduced liquidity, and replaced with new names that meet criteria

> **Exact criteria:** I'm not sure beyond these qualitative descriptions; the index rulebook would be needed.

**Concentrating liquidity:**

- Investors roll into the new on-the-run series; liquidity drops after roll (seen explicitly in index options hedging costs)

### Roll Schedules Conceptually

- The book indicates a six-month roll cadence and illustrates March/September issuance (for 5Y index swaps) with June/December maturities
- Hull describes portfolio updates on March 20 and September 20 for major index portfolios

> **For your specific index (exact dates, business-day adjustments, series naming):** I'm not sure without the rulebook.

### Roll Consequences

**Series basis (on-the-run vs off-the-run) conceptually:**

If on-the-run and off-the-run have different constituents and different time-to-maturity (new is ~6 months longer), their market levels can differ, creating a "series basis" between contracts.

**How rolling affects hedging and P&L explain:**

O'Kane attributes P&L impact from roll to:
1. Composition changes and perceived credit quality
2. Longer maturity by six months, with upward-sloping credit curves tending to widen spreads all else equal

**Liquidity/transaction costs:** on-the-run liquidity generally tighter; off-the-run may have wider bid/ask and hedging mismatch risk.

---

## Defaults and Settlement Inside Indices (Mechanics Only)

### How Default Events of Constituents Affect Index Cashflows

**Premium payments after a constituent default:**

The book's index mechanics reduce outstanding notional by $1/M$ after each default; therefore future coupon cashflows decline proportionally. Constituents that default are removed without replacement.

**Default settlement amount link to auction final price/recovery:**

- O'Kane notes that when deliverables are scarce, ISDA introduced a protocol (2005) allowing a fallback to cash settlement with an auction-determined cash settlement price
- Hull provides a concrete cash-settlement illustration: if auction implies a cheapest deliverable worth 35 per 100, the cash payoff is 65% of notional

> **Note:** We do not derive intrinsic vs quoted index spread calibration here (that's next chapter).

---

## Math and Derivations

### Assumptions (Explicit)

- Coupon payments occur on scheduled dates with accrual $\alpha$ (Actual/360)
- "Upfront" is quoted as a percent of notional
- For cash settlement, "final price" $FP$ is a value per 100 of face; payout follows the single-name cash settlement logic in Hull's example

---

### 6.1 Coupon Payment Formula

Contractual coupon payment per period:

$$\boxed{\text{Payment}_n = N_{\text{out}}(t_n) \times c \times \alpha_n}$$

where:
- $N_{\text{out}}(t_n)$ is outstanding notional at payment date (reduced after defaults)
- $c$ is annual coupon in decimal
- $\alpha_n$ is year fraction under Actual/360

**Unit check:** $N$ in dollars, $c$ dimensionless (per year), $\alpha$ in years $\Rightarrow$ payment in dollars.

**Sanity checks:**
- If $c = 0$, payment is 0
- If $\alpha \approx 0.25$ for a quarter, then quarterly payment $\approx N \times c/4$

---

### 6.2 Upfront Dollar Conversion

If upfront is quoted as $U\%$ of notional:

$$\boxed{U_\$ = N \times U\%}$$

**Unit check:** dollars $=$ dollars $\times$ dimensionless.

---

### 6.3 Accrued Premium (Generic)

If trade occurs between coupon dates $t_{k-1}$ and $t_k$, and $\Delta$ is the elapsed year fraction since $t_{k-1}$:

$$\boxed{\text{Accrued magnitude} = N_{\text{out}} \times c \times \Delta}$$

**Quoting sign convention** (single-name CDS, used generically): accrued is $+\Delta S_0$ for short protection and $-\Delta S_0$ for long protection, scaled by notional.

---

### 6.4 Default Settlement Payoff Mapping from Auction Final Price

Hull's cash settlement example implies:

$$\boxed{\text{Payoff to protection buyer} = N_{\text{name}} \times \left(1 - \frac{FP}{100}\right)}$$

because $FP = 35 \Rightarrow 65\%$ payout.

**Sanity checks:**
- If $FP = 100$, payoff $= 0$
- If $FP = 0$, payoff $= N_{\text{name}}$

---

## Measurement & Risk (Only What Belongs in Chapter 45)

### Index "Spread DV01 / CS01" Concept (Mechanics-First)

**Practical idea:** sensitivity of index PV (or upfront) to a 1 bp shift in the quoted index spread level.

**Precision caveat:** the book explicitly notes PV01 disagreement matters for translating spread quotes into upfront; thus a precise DV01 depends on the valuation convention.

**Working definition (finite difference; assumption):**

$$\text{Spread DV01} \approx \frac{PV(S + 1\text{bp}) - PV(S)}{1\text{bp}}$$

> I'm not sure what your desk's canonical $PV(\cdot)$ convention is (clean vs full, recovery assumption, curve method). You'd need your valuation policy + ISDA/index docs.

### Default Event Exposure (Jump-to-Default Aggregate Effect)

- Each constituent default produces a jump loss (or gain) proportional to its notional share $1/M$ and to $(1 - \text{recovery})$ (book's equal-share mechanics)
- The outstanding notional reduction by $1/M$ also reduces future premium cashflows

### Roll/Series Basis as a Practical Risk (Preview-Level)

- Composition mismatch risk when hedging off-the-run with on-the-run
- Liquidity/hedging cost risk: liquidity drops after roll and can materially affect hedge cost (explicitly noted for index options)

---

## Worked Examples

### Shared Toy Conventions for Examples A–L

| Parameter | Value |
|-----------|-------|
| Notional $N$ | $\$10,000,000$ unless otherwise stated |
| Quarterly coupons | $\alpha = 0.25$ as simple quarterly approximation to Actual/360 (toy) |
| Coupon $c$ | 100 bp $= 0.0100$ (toy; coupon levels vary by series in reality) |

> **Note:** The book's convention is quarterly Actual/360. The book states coupons are chosen near fair value and in 5 bp increments.

---

### Example A: Convert an Index Coupon $c$ into a Quarterly Premium Payment

**Inputs:** $N = \$10{,}000{,}000$, $c = 100$ bp $= 0.0100$, $\alpha = 0.25$

**Step 1:** Compute $N \times c$:

$$\$10{,}000{,}000 \times 0.0100 = \$100{,}000 \text{ per year}$$

**Step 2:** Apply quarterly accrual fraction $\alpha$:

$$\$100{,}000 \times 0.25 = \$25{,}000$$

**Output:** Quarterly premium payment $= \$25{,}000$

**Unit check:** $\$ \times (\text{per year}) \times (\text{years}) = \$$

---

### Example B: Build a Short Coupon Schedule (4 Periods) and Compute Each Payment

**Inputs:** Same as Example A; 4 quarters; $\alpha = 0.25$ each (toy)

**Each quarter payment:**

$$\text{Payment} = 10{,}000{,}000 \times 0.0100 \times 0.25 = \$25{,}000$$

**Total premium over year:**

$$\$25{,}000 \times 4 = \$100{,}000$$

**Output:**

| Period | Payment |
|--------|---------|
| Q1 | $25,000 |
| Q2 | $25,000 |
| Q3 | $25,000 |
| Q4 | $25,000 |
| **Total** | **$100,000** |

---

### Example C: Accrued Premium Between Coupon Dates

**Scenario:** Trade date is halfway through a coupon period.

**Inputs:** $N = \$10{,}000{,}000$, $c = 0.0100$, elapsed fraction $\Delta = 0.125$ (half of a quarter: $45/360$)

**Step 1:** Accrued magnitude:

$$10{,}000{,}000 \times 0.0100 \times 0.125 = 10{,}000{,}000 \times 0.00125 = \$12{,}500$$

**Step 2:** Sign convention (quoting):
- **Long protection:** accrued is negative (you pay it when transferring the position)
- **Short protection:** accrued is positive (you receive it)

**Output:**
- Accrued magnitude: $12,500
- Quoted accrued: $-\$12{,}500$ (long protection) or $+\$12{,}500$ (short protection)

---

### Example D: Upfront Conversion

**Inputs:** Upfront $= 3.25\% = 0.0325$, $N = \$10{,}000{,}000$

**Step:**

$$U_\$ = N \times U\% = 10{,}000{,}000 \times 0.0325$$

$$10{,}000{,}000 \times 0.03 = 300{,}000$$
$$10{,}000{,}000 \times 0.0025 = 25{,}000$$
$$\text{Total} = 325{,}000$$

**Output:** Upfront dollar amount $= \$325{,}000$

**Sign (stated assumption):** We will treat positive upfront as paid by the party entering the position that is "priced below par" (bond-analogy). Actual sign conventions are deal-confirmation specific; I'm not sure without the confirmation.

---

### Example E: Coupon+Upfront Cashflow Timeline

**Inputs:** $N = \$10mm$, $c = 100$ bp, $\alpha = 0.25$, upfront $= +3.25\%$

**Timeline (toy):**

| Time | Cashflow | Amount |
|------|----------|--------|
| $t_0$ | Upfront | $-\$325{,}000$ (paid) |
| $t_1$ (Q1) | Coupon | $-\$25{,}000$ |
| $t_2$ (Q2) | Coupon | $-\$25{,}000$ |
| $t_3$ (Q3) | Coupon | $-\$25{,}000$ |

**Cumulative outflow through $t_3$:**

$$-325{,}000 - 25{,}000 - 25{,}000 - 25{,}000 = -\$400{,}000$$

---

### Example F: Series Basis Toy (On-the-Run vs Off-the-Run Upfront)

**Inputs:** Same coupon; on-the-run upfront $= 2.0\%$, off-the-run upfront $= 2.6\%$, $N = \$10mm$

**Step:** Dollar upfronts

- On-the-run: $10{,}000{,}000 \times 0.020 = \$200{,}000$
- Off-the-run: $10{,}000{,}000 \times 0.026 = \$260{,}000$

**Difference (basis proxy):**

$$\$260{,}000 - \$200{,}000 = \$60{,}000$$

**Interpretation (toy):** Off-the-run is "60k cheaper/more expensive" in upfront terms depending on direction; treat as a simple proxy for series basis (liquidity + composition + maturity effects).

---

### Example G: Roll Trade Mechanics (Close Old Series and Enter New Series)

**Scenario:** You hold long protection in the old series and want to roll to new on-the-run.

**Toy market quotes (upfront paid by protection buyer):**
- Old series upfront = 2.0%
- New series upfront = 1.5%
- Notional $N = \$10mm$

**Step 1:** Unwind old long protection

To close long protection, enter offsetting short protection at market. If protection buyer pays 2.0% to protection seller, then as protection seller you receive:

$$\$10mm \times 0.020 = \$200{,}000$$

**Step 2:** Enter new long protection

Pay upfront:

$$\$10mm \times 0.015 = \$150{,}000$$

**Step 3:** Net upfront exchanged:

$$\$200{,}000 - \$150{,}000 = +\$50{,}000 \text{ (net received)}$$

**Accrued note:** If both trades settle on the same coupon schedule date grid, accrued may largely offset; if not, it won't. Exact schedule rules are index-doc dependent (I'm not sure without the confirmations).

---

### Example H: Liquidity Cost of Rolling (Wider Bid/Ask Off-the-Run)

**Assume bid/ask on upfront:**
- Off-the-run (old): 1.8% / 2.2% (mid 2.0%)
- On-the-run (new): 1.45% / 1.55% (mid 1.50%)

**Rolling a long protection position:**
- Unwind old by selling protection $\Rightarrow$ you receive bid on old
- Enter new long protection $\Rightarrow$ you pay ask on new

**Step 1:** Old unwind slippage vs mid

Mid receipt would be 2.0%; actual receipt is 1.8%.

$$\text{Slippage} = 0.2\% \times \$10mm = 0.002 \times 10{,}000{,}000 = \$20{,}000$$

**Step 2:** New entry slippage vs mid

Mid payment would be 1.50%; actual payment is 1.55%.

$$\text{Slippage} = 0.05\% \times \$10mm = 0.0005 \times 10{,}000{,}000 = \$5{,}000$$

**Output:** Estimated transaction cost of roll $= \$20{,}000 + \$5{,}000 = \$25{,}000$

---

### Example I: Single Default in Index with Auction Final Price $FP = 40$

Use equal-weight notional allocation consistent with the book's mechanics ($1/M$ per name).

**Inputs:** $N = \$10mm$, $M = 125$ (toy but consistent with common examples in the sources for standard portfolios), $FP = 40$

**Step 1:** Per-name notional:

$$N_{\text{name}} = \frac{10{,}000{,}000}{125} = 80{,}000$$

**Step 2:** Cash-settlement payoff magnitude:

$$80{,}000 \times \left(1 - \frac{40}{100}\right) = 80{,}000 \times 0.60 = 48{,}000$$

This matches the "par minus price" logic illustrated by Hull (35 $\Rightarrow$ 65%).

**Output:**
- Payoff to protection buyer: $+\$48{,}000$
- Payoff by protection seller: $-\$48{,}000$

---

### Example J: After-Default Premium Adjustment (Notional Reduction)

The index mechanics reduce outstanding notional by $1/M$ after default.

**Inputs:** $N = \$10mm$, $M = 125$, coupon $c = 0.0100$, $\alpha = 0.25$

**Step 1:** Notional reduction:

$$\Delta N = \frac{10{,}000{,}000}{125} = 80{,}000$$

**Step 2:** New outstanding notional:

$$N_{\text{out}} = 10{,}000{,}000 - 80{,}000 = 9{,}920{,}000$$

**Step 3:** Next-quarter coupon payment before vs after default:

- **Before:** $10{,}000{,}000 \times 0.0100 \times 0.25 = 25{,}000$
- **After:** $9{,}920{,}000 \times 0.0100 \times 0.25$:

$$9{,}920{,}000 \times 0.0100 = 99{,}200$$
$$99{,}200 \times 0.25 = 24{,}800$$

**Output:**
- New coupon payment: $24,800
- Reduction vs pre-default: $25,000 − $24,800 = $200

---

### Example K: Exposure Scaling with Notional ($N = \$5mm$ vs $N = \$50mm$)

Use $c = 0.0100$, $\alpha = 0.25$, upfront $= 3.25\%$

**For $N = \$5mm$:**
- Quarterly coupon: $5{,}000{,}000 \times 0.0100 \times 0.25 = 12{,}500$
- Upfront: $5{,}000{,}000 \times 0.0325 = 162{,}500$

**For $N = \$50mm$:**
- Quarterly coupon: $50{,}000{,}000 \times 0.0100 \times 0.25 = 125{,}000$
- Upfront: $50{,}000{,}000 \times 0.0325 = 1{,}625{,}000$

**Output:** All cashflows scale linearly with $N$.

---

### Example L: On-the-Run Dominance (Toy Hedge Slippage via Bid/Ask)

**Goal:** Quantify incremental cost if hedging in off-the-run with wider bid/ask vs on-the-run.

**Inputs:** Hedge notional $N = \$50mm$
- On-the-run upfront bid/ask: 1.95% / 2.05% (mid 2.00%, half-spread 0.05%)
- Off-the-run upfront bid/ask: 1.60% / 2.00% (mid 1.80%, half-spread 0.20%)

Assume you must cross the spread once (one trade) to put on the hedge.

**Cost approximation:** Transaction cost $\approx$ half-spread $\times N$

- On-the-run cost: $0.05\% \times 50{,}000{,}000 = 0.0005 \times 50{,}000{,}000 = 25{,}000$
- Off-the-run cost: $0.20\% \times 50{,}000{,}000 = 0.0020 \times 50{,}000{,}000 = 100{,}000$

**Incremental cost of using off-the-run:**

$$\$100{,}000 - \$25{,}000 = \$75{,}000$$

**Interpretation (toy):** Wider off-the-run liquidity directly translates into hedge slippage; the book flags liquidity differences and constituent mismatch as practical hedging issues.

---

## Practical Notes

### Booking Checklist for an Index Trade

- [ ] Index family / series (and version if applicable)
- [ ] Maturity (3Y/5Y/7Y/10Y, noting on-the-run tenor is $T \pm 3$ months)
- [ ] Coupon $C$ (fixed contractual spread; check it is the series coupon, multiple of 5 bp in the book's description)
- [ ] Upfront $U\%$ and direction (pay/receive); settlement timing (book: cash settled 3 days after trade date)
- [ ] Notional $N$ and how it maps to per-name notional (often equal share in mechanics: $N/M$)
- [ ] Payment dates, day count (Actual/360)
- [ ] Default settlement assumptions (physical vs cash; auction fallback)
- [ ] Clean vs full MTM / accrued treatment

### Common Pitfalls

1. Mixing "quoted spread" with "coupon+upfront" without a consistent translation (PV01 disagreement is exactly why some indices use bond-price quoting)
2. Forgetting accrued premium when transferring positions between coupon dates
3. Assuming roll dates/coupons/membership rules without sourcing (need rulebook)
4. Confusing "index series" (roll/issuance) with "maturity" (3Y/5Y/7Y/10Y)

### Implementation Pitfalls

- Date schedule generation (coupon dates, settlement lag, roll schedule)
- Unit errors (bp vs decimal; percent vs fraction; price-per-100 vs percent-of-notional)
- Handling defaults consistently: reduce outstanding notional by $1/M$; include accrued-at-default where contract requires it

### Verification Tests

1. **Premium cashflows:** Verify each coupon $= N_{\text{out}} \times c \times \alpha$
2. **Default payoff bounds:** With $FP \in [0, 100]$, payout $N(1 - FP/100) \in [0, N]$
3. **Scaling sanity:** Double notional $\Rightarrow$ double all cashflows

---

## Summary & Recall

### 10-Bullet Executive Summary

1. A CDS index is a standardized basket credit derivative referencing $M$ names; the book models losses and notional reductions in $1/M$ shares
2. Each index series has a fixed constituent list for its life, except defaulted names are removed without replacement
3. Indices roll every six months; market participants often roll positions from old (off-the-run) to new (on-the-run)
4. A "$T$-year" on-the-run index starts near $T+3$ months to maturity and becomes $T-3$ months by the next roll
5. The contractual coupon is fixed, set near fair value at inception and often in 5 bp increments
6. Premium is usually paid quarterly on Actual/360
7. Investment grade indices may quote an "index spread" (flat spread concept); translating spread into upfront can require PV01, which can be disputed
8. Some indices use bond-price style quoting (price − 100) to avoid PV01 translation disputes
9. Defaults trigger settlement (physical or cash via auction fallback) and reduce outstanding notional by $1/M$, shrinking future coupons
10. On-the-run liquidity dominates; liquidity drops after roll, affecting hedge costs (noted explicitly for index options)

---

### Cheat Sheet: Key Definitions & Formulas

| Term/Formula | Definition |
|--------------|------------|
| **Series** | A particular issuance; constituents fixed except default removals |
| **On-the-run** | Most recent series (liquidity benchmark) |
| **Off-the-run** | Older series |
| **Roll** | Transition from old to new series (often by selling old, buying new) |
| **Coupon payment** | $\text{Payment} = N_{\text{out}} \times c \times \alpha$ |
| **Upfront dollars** | $U_\$ = N \times U\%$ |
| **Accrued (magnitude)** | $N \times c \times \Delta$ |
| **Cash-settlement payoff (auction)** | $N(1 - FP/100)$ |

---

## Flashcards (35 Q/A Pairs)

| # | Question | Answer |
|---|----------|--------|
| 1 | What is a CDS index (in one sentence)? | A standardized CDS-like contract referencing a basket of $M$ names, with aggregated premium and default-loss cashflows |
| 2 | What is an index "series"? | A particular issuance of the index with a fixed constituent set for its life (except default removals) |
| 3 | What does "on-the-run" mean for indices? | The most recently issued series that concentrates liquidity (inferred from roll behavior) |
| 4 | What does "off-the-run" mean? | An older series after the next roll; typically less liquid and may have different constituents |
| 5 | How often do indices roll (per the primary source)? | Every six months |
| 6 | Why can rolling cause P&L? | Composition changes and a maturity extension by ~6 months can change perceived credit and spread levels |
| 7 | What is the contractual coupon $C$? | The fixed premium rate specified by the index contract, paid on outstanding notional |
| 8 | How is the coupon chosen at inception (per the book)? | Set close to fair value but usually in multiples of 5 bp |
| 9 | What day count is typical for index coupons in the mechanics description? | Actual/360 |
| 10 | What is the coupon payment formula? | $\text{Payment} = N_{\text{out}} \times c \times \alpha$ |
| 11 | What is "upfront" in indices? | A one-time payment at settlement equal to the contract's value (can be negative/received) |
| 12 | Convert upfront percent to dollars | $U_\$ = N \times U\%$ |
| 13 | What is "index spread" quoting (IG indices)? | A quoted flat-spread level used to price the index like a single CDS, with coupon fixed and upfront implied |
| 14 | Why can spread-quoting be contentious operationally? | Translating spread into upfront requires index PV01, which can be disputed |
| 15 | What's the "bond price convention" for some indices? | Quote a price and compute upfront as (price − 100), avoiding PV01 translation |
| 16 | What happens to constituents within a series when they default? | They are removed without replacement |
| 17 | What happens to outstanding index notional after a default (book mechanics)? | It is reduced by $1/M$ |
| 18 | Why do future coupon payments decline after defaults? | Coupons are paid on outstanding notional, which shrinks by $1/M$ per default |
| 19 | What is accrued premium "clean vs full MTM" about? | Clean MTM excludes accrued to avoid jumps at coupon dates; full includes it |
| 20 | What is the sign of accrued for long protection (quoting convention)? | Negative (you owe accrued when transferring/closing) |
| 21 | What is "coupon accrued at default"? | A contingent cashflow requiring payment of accrued premium up to default time |
| 22 | What is physical settlement in CDS terms? | Protection buyer delivers bonds and receives par from protection seller |
| 23 | What is cash settlement? | Protection seller pays par minus recovery price, determined by dealer poll/auction |
| 24 | Why was an auction protocol important? | It supports cash settlement when deliverables are scarce relative to CDS notional |
| 25 | How does auction final price map to payoff? | Payoff $= N(1 - FP/100)$ (e.g., 35 → 65% payout) |
| 26 | What is "series mismatch risk"? | Hedging off-the-run with on-the-run can mismatch constituents |
| 27 | What does the book say about liquidity and index options after roll? | Liquidity drops after roll, raising hedging costs; thus long-dated expiries beyond six months are rare |
| 28 | What maturities are described as tradable in indices (primary source)? | 3Y, 5Y, 7Y, 10Y, with 5Y and 10Y most liquid |
| 29 | Why does a "5Y" index not start with exactly 5 years maturity? | It starts near 5y+3m and declines to 5y−3m by next roll |
| 30 | What's the operational meaning of "unfunded" trading in indices? | Investors prefer unfunded format; IG indices can quote spread and infer upfront |
| 31 | Why might HY/EM quote as bond price? | To avoid disagreement about index PV01 needed for spread-to-upfront conversion |
| 32 | What is "index basis" (conceptually)? | Quoted index spread can differ from a CDS-implied intrinsic spread, for reasons including contract clause differences and liquidity |
| 33 | Give one clause difference that can create index basis | For one US index, restructuring is excluded (No-Re) while market single-name may use Mod-Re |
| 34 | What is a practical roll P&L driver besides liquidity? | The new series' longer maturity and upward-sloping credit curves can widen the new series spread |
| 35 | What document do you need for exact roll dates, coupons, and membership rules? | The index rulebook (e.g., Markit documentation) plus ISDA definitions/auction protocols |

---

## Mini Problem Set (18 Questions)

*Solution sketches provided for questions 1–9 only.*

---

**1.** Compute the quarterly coupon payment for $N = \$25mm$, $c = 75$ bp, $\alpha = 0.252$.

> **Sketch:** Convert $c = 0.0075$. Payment $= 25{,}000{,}000 \times 0.0075 \times 0.252$.

---

**2.** A trade has upfront $U = 1.80\%$ on $N = \$40mm$. What is $U_\$$?

> **Sketch:** $U_\$ = 40{,}000{,}000 \times 0.018$.

---

**3.** Mid-period elapsed accrual fraction is $\Delta = 0.10$. For $N = \$10mm$, $c = 200$ bp, compute accrued magnitude.

> **Sketch:** $c = 0.02$. Accrued $= 10{,}000{,}000 \times 0.02 \times 0.10$.

---

**4.** Using the quoting sign convention, what is the quoted accrued for a long protection position in (3)?

> **Sketch:** Long protection accrued is negative.

---

**5.** A constituent defaults with auction final price $FP = 25$. Per-name notional is $\$100{,}000$. Compute cash settlement payoff to protection buyer.

> **Sketch:** Payoff $= 100{,}000(1 - 0.25) = 75{,}000$.

---

**6.** With $M = 125$ and index notional $N = \$12.5mm$, compute the notional reduction after one default and new notional outstanding.

> **Sketch:** $\Delta N = N/M$; new $N_{\text{out}} = N - \Delta N$.

---

**7.** Explain two reasons why rolling an index position can generate P&L.

> **Sketch:** Composition change + maturity extension by 6 months.

---

**8.** Give one practical risk when hedging off-the-run with on-the-run.

> **Sketch:** Constituent mismatch across series.

---

**9.** Why might a market prefer bond-price style quoting for some indices?

> **Sketch:** Avoid PV01 disputes in spread-to-upfront conversion.

---

**10.** *(No sketch)* Suppose the old series upfront is 3.0% and the new series is 1.0%. For $N = \$20mm$, compute net cash exchanged for a roll (assume you receive old upfront and pay new upfront).

---

**11.** *(No sketch)* Construct a 4-coupon cashflow table with Actual/360 fractions 0.2528, 0.2611, 0.2528, 0.2528 for $N = \$10mm$, $c = 50$ bp.

---

**12.** *(No sketch)* In a widening market, explain qualitatively why on-the-run might "lead" single-name CDS pricing.

---

**13.** *(No sketch)* Describe the difference between "clean MTM" and "full MTM" and why desks quote clean.

---

**14.** *(No sketch)* An index has two defaults; show the sequence of notional reductions and coupon reductions using $M = 100$, $N = \$10mm$, $c = 100$ bp, $\alpha = 0.25$.

---

**15.** *(No sketch)* Explain why option expiries beyond six months can be illiquid for index options.

---

**16.** *(No sketch)* State what extra documentation you need to be certain about standard coupon levels and settlement conventions for a live trade.

---

**17.** *(No sketch)* Explain why "5Y" indices can start with more than 5 years remaining.

---

**18.** *(No sketch)* Provide a cautious definition of "index spread DV01" and list what assumptions must be specified to compute it.

---

## Source Map

### (A) Verified Facts — Cite Specific Sources

- Index structure, roll mechanics, coupon setting (5 bp increments), Actual/360: O'Kane Ch 9-10
- Cash settlement auction mechanics (35 → 65% example): Hull Ch 25
- Equal-weight loss allocation ($1/M$): O'Kane index mechanics discussion
- March/September roll dates for CDX NA IG and iTraxx Europe: Hull Ch 25
- Liquidity drop after roll affecting index options: O'Kane Ch 10

### (B) Reasoned Inference — Note Derivation Logic

- "On-the-run" as most recent series: inferred from roll description and liquidity concentration language
- Price discovery shifting to upfront with fixed coupon: economic reasoning from contract structure

### (C) Speculation — Flag Uncertainties

- Exact standardized coupon levels for all indices: not asserted as universal in sources
- Exact roll dates, membership rules, settlement conventions for specific indices: requires index rulebook + ISDA definitions
- Desk-specific PV01/DV01 conventions: requires valuation policy documentation
