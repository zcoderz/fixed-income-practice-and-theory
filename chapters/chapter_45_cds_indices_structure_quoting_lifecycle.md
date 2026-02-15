# Chapter 45: CDS Indices — Structure, Quoting, and Lifecycle

---

## Introduction

"CDX.NA.IG.42 5Y" — what does each part of this cryptic string mean, and why does misunderstanding it matter?

When a trader says they are "buying CDX," they usually mean: **buy protection on a standardized portfolio** (one contract instead of 100+ single-name CDS). Indices are popular because they compress a diversified credit view into an operationally simple, liquid instrument—but that simplicity hides three common sources of confusion:

1. **Identifier confusion:** which family/region/quality, which **series**, which **tenor**, and whether the position is **on-the-run** or **off-the-run**.
2. **Quote-object confusion:** is the market quote a **spread** (bp) or a **price/points-upfront** quote? The contract also has a **fixed coupon**, and the quote object determines the **upfront cash** you settle.
3. **Lifecycle confusion:** rolls, defaults, and index factors change the outstanding notional, coupon cashflows, and risk in ways that break booking and P&L if you do not track them explicitly.

**Why this matters:** Consider a trader who books a CDX.NA.HY trade using the wrong quoting convention. High-yield indices quote as "bond price" (not spread), so confusion between the two conventions can cause upfront payment errors of several percent of notional — on a $100 million trade, that's millions of dollars. Similarly, failing to track which series is "on-the-run" versus "off-the-run" affects liquidity, hedge effectiveness, and roll P&L. These aren't theoretical concerns; they're daily operational realities on credit trading desks.

> **Why Middle Office Cares**
>
> Your daily risk report shows CDX.IG DV01 exposure aggregated across the desk. When the index rolls from Series 41 to Series 42, positions need rebooking from the old series to the new. A missed roll booking causes P&L breaks when the old series stops updating and reconciliation issues when positions appear on "dead" instruments. Understanding the roll calendar, the index factor after defaults, and the quoting convention is essential for anyone producing or consuming credit risk reports.

Prerequisites: [Chapter 38 — Single-Name CDS](chapter_38_cds_contract_mechanics.md), [Chapter 40 — CDS Auction Process](chapter_40_cds_auction_process.md), [Chapter 41 — CDS Indices Mechanics (Coupons, Rolls)](chapter_41_cds_indices_mechanics_coupons_rolls.md)  
Follow-on: [Chapter 46 — Intrinsic Index Spread and Index Basis](chapter_46_intrinsic_index_spread_and_index_basis.md), [Chapter 47 — Hedging and Relative Value in CDS Indices](chapter_47_hedging_relative_value_cds_indices.md)

## Learning Objectives
- Decode CDS index identifiers (family/region/quality/series/tenor) and recognize on-the-run vs off-the-run.
- Translate a quote into cash: **quote object → upfront + running coupon cashflows** with explicit units and sign.
- Track what changes after defaults (outstanding notional/factor, premium payments, and default settlement).
- Compute and interpret a simple index **CS01** with an explicit bump object (1bp = \(10^{-4}\)), units, and sign.

**Chapter roadmap:** This chapter focuses on **market structure and operational mechanics**:
- Index naming and identifiers (series/tenor/on-the-run).
- Quoting and settlement cash (spread-quoted vs price-quoted indices).
- Lifecycle mechanics (rolls and what changes).
- Default handling inside an index (notional reduction, auction settlement, accrued premium).
- A short preview of intrinsic spread / index basis (then hand off to Chapter 46).

---

## Index Naming: Decoding the Identifier

A CDS index identifier encodes multiple pieces of information in a structured format. Understanding this structure is essential for correct trade booking and risk management.

### 45.1.1 The Standard Index Families

An index family defines (i) a reference-entity universe (region/sector), (ii) a credit-quality bucket (IG/HY/crossover), and (iii) a ruleset for rolling and maintaining constituents. Table 45.1 is a simplified map of common families.

**Table 45.1: Major CDS Index Families**

| Index Name | Geography | Credit Quality | Number of Credits |
|------------|-----------|----------------|-------------------|
| CDX.NA.IG | North America | Investment Grade | 125 |
| CDX.NA.HY | North America | High Yield | 100 |
| CDX.NA.XO | North America | Crossover | 35 |
| CDX.EM | Emerging Markets | Sovereign | 15 |
| CDX.EM.Diversified | Emerging Markets | Sovereign + Corporate | 40 |
| iTraxx Europe | Europe | Investment Grade | 125 |
| iTraxx Europe Crossover | Europe | Crossover | 40 |
| iTraxx Japan | Japan | Investment Grade | 50 |
| iTraxx Asia ex-Japan | Asia ex-Japan | Investment Grade | 50 |
| iTraxx Australia | Australia | Investment Grade | 25 |

> **Note (counts drift within a series):** Constituent counts typically start at a standard launch size, then decline within a series as defaults remove names (without replacement).

> **Crossover:** “Crossover” indices target credits near the IG/HY boundary (often fallen angels / crossover-rated names).

> **Desk Reality:** Many screens show a “generic” index level (auto-rolling) and also quote specific series.
> **Common break:** Booking the generic while risk/P&L is tracked on a specific series creates breaks at roll (and after defaults when the factor changes).
> **What to check:** Record the exact series and the current factor from confirmation/reference data for every trade.

### 45.1.2 Index Governance and Administration

Index composition is maintained by an independent administrator under published rules (typically with dealer/investor input). Operationally, you treat the administrator’s reference data as the source of truth for:
- the official constituent list for each series,
- the fixed coupon for that series,
- the current index factor / outstanding notional after defaults,
- and any published corporate-action updates that affect trade processing.

### 45.1.3 Anatomy of an Index Identifier

A complete index identifier follows the pattern:

$$\boxed{\text{[Family].[Region].[Quality].[Series]}\ [\text{Tenor}]}$$

**Example: CDX.NA.IG.42 5Y**

| Component | Value | Meaning |
|-----------|-------|---------|
| CDX | — | Index family (North American indices) |
| NA | — | North America |
| IG | — | Investment Grade |
| 42 | — | Series number (42nd vintage) |
| 5Y | — | 5-year maturity contract |

**Example: iTraxx Europe Series 6 7Y**

| Component | Value | Meaning |
|-----------|-------|---------|
| iTraxx | — | Index family (European indices) |
| Europe | — | European constituents |
| (IG implied) | — | Investment grade unless otherwise noted |
| Series 6 | — | 6th vintage |
| 7Y | — | 7-year maturity contract |

### 45.1.4 Why Indices Exist: Operational Advantages

CDS indices exist because they turn “a view on broad credit” into a *single, standardized trade*:

1. **Diversified exposure in one ticket:** one contract references a large portfolio instead of dozens of single-name trades.
2. **Efficient hedging:** a straightforward macro hedge for credit books when single-names are illiquid or operationally costly.
3. **Standardized plumbing:** fixed coupon + upfront settlement makes trades easier to compare and net.
4. **Building block for other products:** standardized indices are common underlyings for tranches and index options.
5. **Relative value across buckets:** index families and sub-indices let you express IG vs HY (or sector) views with cleaner execution.

> **Desk Reality:** Liquidity is usually deepest in the on-the-run benchmark tenors; off-the-run series can remain tradable but can behave like a different instrument (different constituents, different roll dynamics).

### 45.1.5 Series and Version: The Vintage System

Major index families roll on a schedule (commonly semi-annually). Each roll creates a new **series** with:
- a **fixed coupon** for the life of that series, and
- a **fixed constituent list** for that series (except that defaults remove names without replacement).

**Series numbering:**
- The series number increments each roll for a given index family.
- The exact "current" series depends on the date; always book the series specified on the confirmation (or the official index documentation).

**On-the-run vs. off-the-run:**
- **On-the-run:** the most recently launched series (typically the liquidity benchmark).
- **Off-the-run:** earlier series (still referenced and often tradable, but can be less liquid and may have different constituents).

### 45.1.6 Maturity Labels and Actual Tenors

Tenor labels (3Y/5Y/7Y/10Y) refer to **standard maturity dates**, not “exactly T years from today.” Because series are launched only on specific roll dates, the on-the-run “5Y” at launch can have *slightly more* than five years remaining, and it “ages” into *slightly less* than five years remaining by the next roll.

**Rule of thumb (±3 months):**
- at issuance, an on-the-run “T-year” index is often around \(T+3\) months to maturity,
- six months later (at the next roll), it is around \(T-3\) months to maturity.

**Typical maturity alignment (check the confirmation):**
- a March roll can align to a June maturity date for the corresponding tenor,
- a September roll can align to a December maturity date for the corresponding tenor.

**Table 45.2: Standard Maturity Points**

| Maturity Label | Most Liquid? | Typical Use |
|----------------|--------------|-------------|
| 3Y | No | Short-dated hedging |
| 5Y | Yes | Benchmark trading, hedging |
| 7Y | No | Curve trades |
| 10Y | Yes | Long-dated exposure |

Practical intuition: the 5Y tenor is often treated as the benchmark point; other tenors can exist but may be less liquid depending on the family and market conditions.

---

## Quoting: Spread vs. Price

One of the most operationally critical aspects of CDS indices is understanding when they quote as **spread** versus when they quote as **price**. Getting this wrong leads to booking errors.

### 45.2.1 The Two Quoting Regimes

Most CDS indices trade with a **fixed running coupon** plus an **upfront** amount at initiation (“points upfront”). The market then uses one of two quote objects:

- **Index spread quote** \(S_{I,\text{bp}}\) (bp per year): commonly used for investment-grade indices (e.g., CDX IG, iTraxx Europe). The quoted spread is interpreted as a *flat* spread level at which the index would be priced under a standard valuation convention.
- **Bond-price quote** \(P_{\text{bond}}\) (price per 100 notional): commonly used for lower credit-quality indices (e.g., HY and some EM/crossover families). The quoted price encodes “points upfront” directly via \(U_{\%}=100-P_{\text{bond}}\) under the sign convention defined in this chapter.

A practical reason for the price convention is that it avoids disagreements about the PV01 assumptions needed to convert a spread quote into an upfront cash amount.

**Table 45.3: Quoting Convention by Index Family (Coupon is Series-Specific)**

| Index Family (examples) | Common Quote Object | Fixed Coupon | Practical Motivation |
|--------------|---------------------------|--------------|--------|
| CDX.NA.IG, iTraxx Europe | Spread | Fixed at series inception | Convert spread → upfront via PV01 |
| CDX.NA.HY, CDX.EM, iTraxx Crossover | Price | Fixed at series inception | Quote upfront directly (avoid PV01 disputes) |

> **Desk Reality: "Points Upfront" Language**
>
> **Desk Reality:** Traders often speak in “over/under” relative to the fixed coupon (e.g., “50 over” means the quoted spread is 50 bp above coupon).
> **Common break:** People forget “over/under” is *relative to the series coupon* and flip the upfront sign.
> **What to check:** Compute the sign from the inequality \(S_{I,\text{bp}} \gtrless C_{\text{bp}}\): if \(S>C\) the protection buyer pays upfront; if \(S<C\) the buyer receives upfront.
>
> Example translations:
> - “50 over” with a 100 bp coupon means \(S_{I,\text{bp}}=150\) bp.
> - “20 under” with a 100 bp coupon means \(S_{I,\text{bp}}=80\) bp.

> **Pitfall — spread vs price quote:** The identifier alone does not tell you the quote object (bp spread vs price per 100).
> **Why it matters:** You will book the wrong upfront (often by multiple % of notional) and create immediate cash/P&L breaks.
> **Quick check:** Check the **unit label**. If you can convert with \(U_{\%}=100-P_{\text{bond}}\), you are looking at a **price per 100** quote; if you need a coupon and PV01 to convert, you are looking at a **spread (bp)** quote.

### 45.2.2 Spread Quoting (Many Investment-Grade Indices)

When an index quotes by spread, the screen quote is an annualized **index spread** \(S_{I,\text{bp}}\) in bp/year, while the contract has a fixed running coupon \(C_{\text{bp}}\) in bp/year. The operational translation is:

- spread difference \((S_{I,\text{bp}}-C_{\text{bp}})\) (bp/year)
- times index risky PV01 \(\text{RPV01}_I\) (currency per bp)
- equals upfront dollars \(U_{\$}\) (currency)

Let \(N_{\text{out}}\) be the current outstanding notional (after defaults). Define \(U_{\$}>0\) to mean “protection buyer pays upfront.” A common approximation is:

$$\boxed{U_{\$} \approx (S_{I,\text{bp}}-C_{\text{bp}})\,\text{RPV01}_I}$$

**Anchor (fixed-coupon regime):** the upfront is the present value of the difference between payments at the quoted spread and payments at the fixed coupon for the remaining life of the transaction (estimated under a specified procedure). If the quoted spread is **less** than the coupon, the **seller** of protection pays this PV to the buyer at trade inception; if the quoted spread is **greater** than the coupon, the **buyer** pays it to the seller.

**Expand (replication story):** treat the screen spread \(S_I\) as the “par running premium” that would clear the index with **zero** upfront under your valuation convention. Trading at the fixed coupon \(C\) changes the PV of the premium leg by roughly \((S_I-C)\times \text{RPV01}_I\). The upfront is the one-time cash exchange that offsets that PV difference at initiation.

**Check (limiting cases):**
- If \(S_I=C\), then \(U_{\$}\approx 0\): the fixed coupon happens to be par.
- If \(C=0\), then \(U_{\$}\approx S_I \cdot \text{RPV01}_I\): you are effectively paying (all) the premium PV upfront to buy protection with no running coupon.

Convert to “points upfront” (percent of notional) via:

$$\boxed{U = \frac{U_{\$}}{N_{\text{out}}}, \qquad U_{\%}=100\,U}$$

**Useful identity (unit check):** if you work with the risky annuity \(A_I\) in years (Chapter 38), then
\(\text{RPV01}_I = N_{\text{out}} \cdot 10^{-4} \cdot A_I\). In that case,
\(U \approx (S_I-C)\,A_I\) when \(S_I,C\) are in decimal form (e.g., 160 bp = 0.0160).

**Interpretation (with this sign):**
- If \(S_{I,\text{bp}} > C_{\text{bp}}\): protection buyer **pays** upfront (index “below par”)
- If \(S_{I,\text{bp}} < C_{\text{bp}}\): protection buyer **receives** upfront (index “above par”)

**Flat-curve convention (important):** In many market implementations, the quoted index spread is treated as a **flat spread curve** for valuation (one spread level used to compute the PV01 for that maturity). This means the PV01 used in the spread→upfront conversion is tied to a particular convention, not a universal truth.

### 45.2.3 Bond Price Quoting (High Yield / EM)

For some index families, the market quotes a **price per 100 notional** so the upfront is read off directly.

$$\boxed{U_{\%} = 100\% - P_{\text{bond}}}$$

**Examples:**
- Bond price = 102.18% ⇒ \(U_{\%}=100-102.18=-2.18\%\). Under our sign convention, the **protection buyer receives** 2.18% upfront (equivalently, the long-credit side pays 2.18%).
- Bond price = 99.50% ⇒ \(U_{\%}=100-99.50=+0.50\%\). Under our sign convention, the **protection buyer pays** 0.50% upfront (equivalently, the long-credit side receives 0.50%).

**Dollar upfront from bond price (use outstanding notional):**

$$\boxed{U_{\$} = N_{\text{out}} \times (1 - P_{\text{bond}}/100)}$$

**Check (points → dollars):** 1.00 point is 1% of notional. So a 0.01-point move corresponds to \(N_{\text{out}}\times 0.01\%\) dollars (e.g., on \(N_{\text{out}}=\$50\text{mm}\), 0.01 point is \(\$5{,}000\)). This is a fast reconciliation check for price-quoted indices.

**Why this exists:** spread→upfront conversion requires a PV01 convention (and sometimes iteration). A price quote sidesteps that by making the upfront the quote object.

### 45.2.4 Converting Between Spread and Price

**From spread to upfront dollars (spread-quoted):**

Given \(S_{I,\text{bp}}\), \(C_{\text{bp}}\), and \(\text{RPV01}_I\) (currency per bp):

$$\boxed{U_{\$} \approx (S_{I,\text{bp}}-C_{\text{bp}})\,\text{RPV01}_I}$$

**From upfront dollars to implied spread (hold the PV01 convention fixed):**

$$\boxed{S_{I,\text{bp}} \approx C_{\text{bp}} + \frac{U_{\$}}{\text{RPV01}_I}}$$

To get **percent-of-notional**, use \(U_{\%} = 100\,U_{\$}/N_{\text{out}}\). To get the **price per 100**, use:
$$\boxed{P_{\text{bond}} = 100 - U_{\%}.}$$

### 45.2.5 The Index Factor

When defaults occur, the index notional reduces. The **index factor** tracks this:

$$\boxed{f = \frac{M - D}{M}}$$

where $M$ is the original constituent count and $D$ is the number of defaulted names.

**Examples:**
- CDX.IG with 0 defaults: $f = 125/125 = 1.000$
- CDX.IG with 2 defaults: $f = 123/125 = 0.984$
- CDX.HY with 5 defaults: $f = 95/100 = 0.950$

**Outstanding notional:**

$$\boxed{N_{\text{out}} = N \times f}$$

> **Desk Reality:** The index administrator publishes the official index factor / outstanding notional after credit events.
> **Common break:** A stale factor makes notional, coupon cashflows, and risk (DV01/CS01) wrong and creates cash/P&L breaks after defaults.
> **What to check:** Confirm the factor used in risk and booking against the official factor for the specific series.

### 45.2.6 Clean vs. Full (Dirty) Pricing

The quote you see on the screen is usually **clean** in the bond sense: it is focused on the “economic” value (spread vs coupon) and excludes the mechanics of **accrued premium**.

At trade entry, two cash amounts matter:
1. **Upfront** implied by the quote object (Sections 45.2.2–45.2.4).
2. **Accrued premium** from the previous coupon date up to the effective date (typically paid by the protection buyer to the protection seller).

A simple accrued-premium magnitude is:

$$\boxed{\text{Accrued Premium} = N_{\text{out}} \times C \times \Delta(t_{\text{last coupon}}, t_{\text{effective}})}$$

where \(\Delta(\cdot,\cdot)\) is a day-count year fraction (often ACT/360 for standard CDS coupons; always confirm for the specific index).

**Check (accrued scale):** one day of accrued premium is about \(N_{\text{out}}\times C \times (1/360)\). For \(N_{\text{out}}=\$50\text{mm}\) and \(C=100\) bp, that is \(\$13{,}889\) per day. If your accrued is off by 10×, you likely mixed bp and percent or used the wrong day-count denominator.

> **Quick check:** If one system reports “clean” and another reports “full/dirty,” the difference is often just accrued premium (plus small settlement mechanics). When P&L breaks, reconcile **upfront vs accrued** before assuming the model is wrong.

### 45.2.7 Risk Mapping: CS01 (What Is Being Bumped?)

This book’s convention is to define risk scalars using a **down bump**:

$$\boxed{\text{CS01} := PV(\text{spread down }1\text{bp})-PV(\text{base})}$$

To make this unambiguous, always state:
- **Bump object:** here, bump the **spread input used to price the index** (often the quoted flat index spread under the valuation convention).
- **Bump size:** \(1\text{bp} = 10^{-4}\).
- **Units:** currency per 1bp for the stated notional (and current factor).
- **Sign:** for **long protection**, spread-down is good news (tighter credit), so CS01 is typically **negative**; for **short protection**, it is typically positive.

**First-order check (near par):** holding the PV01 convention fixed,
\[
\text{CS01} \approx -\text{RPV01}_I
\]
for **long protection**, where \(\text{RPV01}_I\) is the PV of 1 bp/year of running premium for the position (currency per bp). Because \(\text{RPV01}_I \propto N_{\text{out}} = Nf\), CS01 scales roughly linearly with the index factor.

**Convention bridge (up-bump vs down-bump):** you will also see an **up-bump** convention,
\(\text{CS01}^{\uparrow} := PV(\text{spread up }1\text{bp})-PV(\text{base})\),
under which **long protection is typically positive**. For small bumps under the same “held fixed” assumptions, the two are approximately related by a sign flip:
\[
\text{CS01}^{\uparrow} \approx -\text{CS01}.
\]
The number is only meaningful once you pin down the bump object (spread vs price), the curve rebuild rule, and the sign convention used by your risk reports.

---

## 45.3 Index Lifecycle: From Launch to Maturity

### 45.3.1 Series Launch and Roll Dates

Every 6 months—commonly on **March 20** and **September 20** for the best-known families (e.g., iTraxx and CDX)—the indices roll and a new series starts trading. Maturity dates for CDS-style contracts are typically aligned to the 20th-of-month grid (March/June/September/December).

**The roll calendar:**

| Issue Date | Maturity Alignment | Next Roll |
|------------|-------------------|-----------|
| March 20 | Matures June 20 (T years out) | September 20 |
| September 20 | Matures December 20 (T years out) | March 20 |

### 45.3.2 What Happens at Roll

Investors who hold the existing on-the-run index and want to stay in the benchmark instrument generally roll into the new on-the-run index by **selling the old contract and buying the new one**.

> **Analogy: The Rolling Train**
>
> Imagine a train (Liquidity) that switches tracks every 6 months.
>
> *   **On-the-Run (e.g., Series 42)**: The train is here. Bid-offer is tight and there are more two-way markets.
> *   **The Roll (Mar 20)**: The train switches to the Series 43 track.
> *   **The Choices**:
>     1.  **Roll**: Jump to Series 43 (Pay transaction costs). Stay liquid.
>     2.  **Stay**: You are now "Off-the-Run." The station is emptier. Bid-offer is wider and exits can be more costly.
> *   **The Force**: Many participants roll to stay in the most liquid instrument, so activity often concentrates around Mar 20 / Sep 20.

> **Desk Reality:** Roll periods concentrate liquidity and operational risk. Desks often quote a “roll level” (old series vs new series), and product control cares because any missed rebooking creates reconciliation breaks.

Rolling can generate P&L from (at least) two structural sources:

1. **Composition changes:** the new series can drop names (downgrades, illiquidity, corporate actions) and add replacements, so it is not the same portfolio.
2. **Maturity extension:** the new on-the-run series typically extends maturity by about six months versus the old series, which changes fair spreads and PV01 even if constituents were identical.

### 45.3.3 Constituent Changes Between Series

Within a series, the constituent list is intended to be fixed. If a name defaults, it is removed (and the index’s outstanding notional is reduced accordingly).

**Across series**, the constituent list can change due to:
- Downgrade out of investment grade
- Credit events
- Material reduction in liquidity
- New qualifying names added as replacements

### 45.3.4 Settlement Timing

Operationally, distinguish:
- **trade date** (you agree the trade),
- **effective date** (protection starts; accrual clock),
- **cash settlement date** (upfront + accrued premium exchanged).

| Event | Timing |
|-------|--------|
| Trade date | T |
| Cash settlement | T + (platform/confirmation-specific) |
| First coupon | Next scheduled coupon date |
| Subsequent coupons | Quarterly (day count per contract) |

**Coupon setting at inception:** each series has a fixed running coupon specified by the index terms. In the standardized fixed-coupon regime, the coupon is chosen from a small set of standard coupons (e.g., 100 bp for many IG indices, 500 bp for many HY indices) and the upfront adjusts so the trade clears at the market level. Always use the coupon specified for the series you are trading.

---

## 45.4 Default Handling Within Indices

When a constituent defaults, the index contract must handle the credit event systematically. This section covers the mechanics; Chapter 40 provides the detailed auction process.

### 45.4.1 What Happens When a Constituent Defaults

At a high level, a default inside an equal-weight index does three things:

1. **Protection payment on that name’s slice:** the defaulted name represents \(1/M\) of the original index notional, so protection payment is computed on a per-name notional of \(N/M\) (settled via the standard CDS settlement process, often an auction; see Chapter 40).
2. **Accrued premium up to default:** premium is accrued up to the default date and is exchanged as part of settlement.
3. **Outstanding notional shrinks:** after settlement, the index stops paying/receiving premium on the defaulted name, so future premium payments (and spread risk) are reduced.

### 45.4.2 Default Settlement Timeline

The detailed timeline is handled by the CDS auction/settlement framework (Chapter 40). Conceptually:

| Stage | What happens |
|---|---|
| Credit event occurs | A credit event is triggered for a constituent under the index’s contractual credit-event definition. |
| Event determination | A determination process confirms whether the event is a credit event for that contract. |
| Auction | An auction produces a final price used for cash settlement. |
| Settlement | Cashflows are exchanged: protection payment and accrued premium. |

### 45.4.3 Cash Settlement Calculation

From the auction final price $FP$ (value per 100 of deliverable), the protection payment is:

$$\boxed{\text{Protection Payment (per name)} = \frac{N}{M} \times \left(1 - \frac{FP}{100}\right)}$$

**Example:** If the final price is 35 per 100, the cash payoff is \(65\%\) of the per-name notional.

### 45.4.4 Accrued Premium at Default

The accrued premium on the defaulted credit is paid:

$$\text{Accrued at Default} = \frac{N}{M} \times C \times \Delta(t_{\text{last coupon}}, \tau)$$

where $\tau$ is the default time.

### 45.4.5 Notional Reduction After Default

After settlement, the index outstanding notional reduces:

$$\boxed{f^{\text{new}} = f^{\text{old}}-\frac{1}{M}, \qquad N_{\text{out}}^{\text{new}} = N\cdot f^{\text{new}} = N_{\text{out}}^{\text{old}}-\frac{N}{M}}$$

For a 125-name index, each default reduces the factor by \(1/125 = 0.008\) (0.8 percentage points) and reduces outstanding notional by \(N/125\).

> **Deep Dive: The "Big Short" Mechanics**
>
> In the movie *The Big Short*, Dr. Burry waits for defaults. How does the payout actually look?
>
> 1.  **The Wait**: He pays premium (bleed). The index notional is full (\$100m).
> 2.  **The Event**: A name defaults.
> 3.  **The Payout**: He gets paid $(1-R) \times \$800k$ immediately (cash settlement).
> 4.  **The Aftermath**: The index notional shrinks to \$99.2m. He stops paying premium on the dead name.
> 5.  **The End Game**: If 10 names default, he collects $\sim\$6m$ in payouts, and the index shrinks. He doesn't get "The Jackpot" all at once; he gets it name-by-name interaction.

**Impact on future coupons:**

$$\text{Coupon}_{\text{new}} = N_{\text{out}}^{\text{new}} \times C \times \alpha$$

where $\alpha$ is the accrual fraction for the period.

### 45.4.6 Regional Differences in Credit Event Definitions

Credit-event definitions are **contract-specific** and can differ across index families/regions (for example, whether “restructuring” is a credit event).

**Table 45.4: Credit Event Definitions by Region**

| Index Family | Bankruptcy | Failure to Pay | Restructuring | Convention Name |
|--------------|------------|----------------|---------------|-----------------|
| CDX.NA.IG | Yes | Yes | **No** | No-Re (XR) |
| CDX.NA.HY | Yes | Yes | **No** | No-Re (XR) |
| iTraxx Europe | Yes | Yes | **Yes** (Mod-Mod-Re) | MM |
| iTraxx Crossover | Yes | Yes | **Yes** (Mod-Mod-Re) | MM |

**Basis implications:**

This creates structural basis between indices and single-name CDS:
- If single-name contracts trade with Mod-Re but the index is No-Re, there's a mismatch
- A restructuring event triggers single-name CDS but NOT the CDX index
- Hedgers must account for this "restructuring basis" when using indices to hedge single-name exposure

> **What to check:** do not assume the index and your single-name hedge use the same credit-event definition. Confirm the restructuring clause (and other credit-event terms) on the index termsheet / confirmation.

---

## 45.5 Intrinsic Spread and Index Basis: Preview

Chapter 46 provides the detailed treatment of intrinsic spread calculation and index basis trading. Here we preview the core concepts since they are essential for understanding index valuation.

### 45.5.1 The Intrinsic Spread Concept

The **intrinsic spread** of an index is the theoretical spread implied by its constituents. A common approximation is an RPV01-weighted average of constituent spreads:

$$\boxed{S_I^{\text{intr}}(t,T) \approx \frac{\sum_{m=1}^{M} S_m(t,T) \cdot \text{RPV01}_m(t,T)}{\sum_{m=1}^{M} \text{RPV01}_m(t,T)}}$$

where:
- $S_m$ = single-name spread for constituent $m$
- $\text{RPV01}_m$ = risky PV01 for constituent $m$

**Why RPV01-weighted, not equal-weighted?**

A simple average of constituent spreads would be incorrect because each constituent has different credit risk, hence different RPV01. The intrinsic spread must be weighted by the amount of premium each name contributes to the index.

### 45.5.2 Index Basis

The **index basis** is the difference between the quoted index spread and the intrinsic spread:

$$\boxed{\text{Index Basis} = S_I^{\text{quoted}} - S_I^{\text{intrinsic}}}$$

Why the basis can be positive or negative (mechanism-level):

1. **Contract mismatch (e.g., restructuring clause):** if the index and single-names are not economically identical contracts, their spreads need not match.

2. **Liquidity premium:** Index liquidity may command a premium (tighter spread) versus single-name average.

3. **Price discovery / lead-lag:** the index can move before (or after) single-name spreads adjust, creating temporary basis.

**Chapter 46** covers basis calculation in full detail, including worked examples and trading strategies.

---

## 45.6 Worked Examples

### Example 45.A: Parse a Complete Index Identifier

**Task:** Decode "CDX.NA.HY.7 5Y" and identify all relevant parameters.

**Solution:**

| Component | Parsed Value | Meaning |
|-----------|--------------|---------|
| CDX | Index family | North American CDS index family |
| NA | Region | North America |
| HY | Credit quality | High Yield |
| 7 | Series | 7th vintage (series numbering; date depends on index family) |
| 5Y | Tenor label | 5-year maturity contract |

**Additional facts from Table 45.1:**
- Number of credits: 100 (standard for CDX.NA.HY)
- Quoting convention: Bond price (not spread)
- Fixed coupon: Fixed at series inception (series-specific; do not assume)
- Roll dates: March 20 and September 20

---

### Example 45.B: Convert Between Spread and Price Quoting

**Example Title:** From a spread quote to settlement cash (and a CS01 check)

**Context**
- You are booking a **buy-protection** index trade and need the **cash settlement** and a first-pass **CS01**.

**Timeline (Concrete Dates)**
- Trade date: 2026-02-16
- Effective date (assumed for this example): 2026-02-17
- Cash settlement date (assumed for this example): 2026-02-20
- Previous coupon date: 2025-12-20
- Next coupon date: 2026-03-20

**Inputs**
- Instrument: 5Y investment-grade index (series per confirmation)
- Notional: \(N=\$50{,}000{,}000\)
- Current index factor: \(f=0.984\) ⇒ \(N_{\text{out}}=Nf=\$49{,}200{,}000\)
- Fixed coupon: \(C=100\text{bp}=0.0100\)
- Quoted index spread: \(S_{I,\text{bp}}=160\text{bp}\) (i.e., \(S_I=0.0160\))
- Risky annuity: \(A_I=4.20\) years (per unit notional, per Chapter 38)
- Day count for coupon accrual: ACT/360

**Outputs (What You Produce)**
- Upfront (dollars): \(U_{\$}\approx (S_{I,\text{bp}}-C_{\text{bp}})\text{RPV01}_I\)
- Upfront (fraction): \(U=U_{\$}/N_{\text{out}}\)
- Upfront (percent): \(U_{\%}=100U\)
- Price per 100 (if needed): \(P_{\text{bond}}=100-U_{\%}\)
- Accrued premium at entry
- CS01 (first-order check; bump object defined)

**Step-by-step**
1. Translate quote → upfront:
   - Convert annuity to \(\text{RPV01}\) (currency per bp for this position):
     \(\text{RPV01}_I = N_{\text{out}} \times 10^{-4} \times A_I = \$49.2\text{mm}\times 10^{-4}\times 4.20 = \$20{,}664/\text{bp}\)
   - Spread difference: \(S_{I,\text{bp}}-C_{\text{bp}} = 160-100 = 60\) bp
   - Dollar upfront \(U_{\$}\approx 60\times \$20{,}664 = \$1{,}239{,}840\) (paid by protection buyer)
   - Upfront fraction \(U=U_{\$}/N_{\text{out}} = 1{,}239{,}840/49.2\text{mm} = 0.0252\) ⇒ \(U_{\%}=2.52\%\)
2. Accrued premium from last coupon to effective date:
   - Day count: \(\Delta = 59/360 \approx 0.163889\) (2025-12-20 → 2026-02-17)
   - Accrued premium \(=N_{\text{out}}\times C\times \Delta=\$49.2\text{mm}\times 0.0100\times 0.163889=\$80{,}633.33\) (paid by protection buyer)
3. Settlement cash (what actually moves on settlement date):
   - Total cash at settlement \(\approx \$1{,}239{,}840+\$80{,}633.33=\$1{,}320{,}473.33\) paid
4. CS01 check (spread down 1bp, hold other inputs fixed):
   - With the down-bump definition, \(\text{CS01}\approx -\text{RPV01}_I = -\$20{,}664/\text{bp}\) for long protection.

**Cashflows (illustrative)**
| Date | Cashflow | Explanation |
|---|---:|---|
| 2026-02-20 | \(-\$1{,}239{,}840\) | Upfront implied by spread vs coupon |
| 2026-02-20 | \(-\$80{,}633.33\) | Accrued premium (2025-12-20 → 2026-02-17) |
| 2026-03-20 | \(-\$123{,}000\) | Next running coupon (assumes no further defaults; \(0.25\times N_{\text{out}}\times C\)) |

**P&L / Risk Interpretation**
- The settlement cash is **upfront + accrued**. If you reconcile cash but only one leg is booked, you will see an immediate P&L break.
- CS01 sign depends on your definition; here, with the down-bump convention, **long protection** typically has **negative** CS01.

**Sanity Checks**
- Units: (bp) × (currency per bp) = currency ✓
- Units (alt): (decimal spread) × (years) = fraction of notional ✓
- Limit check: if \(S_I=C\), then \(U=0\) and price is ~100 ✓
- Factor check: if \(f\) decreases, upfront dollars and coupons scale down linearly ✓

**Debug Checklist (When Your Result Looks Wrong)**
- Did you treat the quote as **spread** vs **price** correctly?
- Did you use the correct **series coupon** (do not assume 100/500 without checking)?
- Did you apply the **current factor** \(f\) to notional and coupons?
- Are you mixing “clean” (upfront) with “full” (upfront + accrued) in the cash reconciliation?

---

### Example 45.C: Calculate Index Notional and Coupon After Defaults

**Given:** CDX.NA.IG with:
- Initial notional $N = \$100,000,000$
- Number of constituents $M = 125$
- Coupon $C = 100$ bp $= 0.0100$
- Quarterly accrual fraction $\alpha = 0.25$
- **Two defaults have occurred**

**Task:** Calculate outstanding notional, index factor, and quarterly coupon.

**Step 1:** Index factor:

$$f = \frac{M - D}{M} = \frac{125 - 2}{125} = \frac{123}{125} = 0.984$$

**Step 2:** Outstanding notional:

$$N_{\text{out}} = N \times f = 100,000,000 \times 0.984 = \$98,400,000$$

**Step 3:** Quarterly coupon payment:

**Before defaults:**
$$\text{Coupon} = 100,000,000 \times 0.0100 \times 0.25 = \$250,000$$

**After 2 defaults:**
$$\text{Coupon} = 98,400,000 \times 0.0100 \times 0.25 = \$246,000$$

**Reduction:** $\$250,000 - \$246,000 = \$4,000$ per quarter

---

### Example 45.D: Default Settlement Calculation

**Given:**
- Index notional $N = \$50,000,000$
- Number of names $M = 125$
- One constituent defaults
- Auction final price $FP = 35$ (per 100 face value)

**Task:** Calculate the protection payment.

**Step 1:** Per-name notional:

$$N_{\text{name}} = \frac{N}{M} = \frac{50,000,000}{125} = \$400,000$$

**Step 2:** Loss given default (LGD) fraction:

$$\text{LGD fraction} = 1 - \frac{FP}{100} = 1 - 0.35 = 0.65 = 65\%$$

**Step 3:** Protection payment:

$$\text{Payment} = N_{\text{name}} \times \text{LGD fraction} = 400,000 \times 0.65 = \$260,000$$

**Sanity checks:**
- If $FP = 0$ (total loss): Payment $= \$400,000$ ✓ (bounded by per-name notional)
- If $FP = 100$ (full recovery): Payment $= \$0$ ✓

---

### Example 45.E: Roll Trade Mechanics

**Scenario:** You hold long protection on off-the-run CDX.NA.IG Series 6 and want to roll to on-the-run Series 7.

**Given:**
- Notional $N = \$25,000,000$
- Series 6 (off-the-run) upfront to unwind: you receive 0.80% to close
- Series 7 (on-the-run) upfront to enter: you pay 0.50%

**Task:** Calculate net cash from roll.

**Step 1:** Cash from unwinding Series 6:

To close long protection, you enter offsetting short protection. You receive:
$$\text{Receive} = 25,000,000 \times 0.0080 = \$200,000$$

**Step 2:** Cash to enter Series 7:

To enter new long protection, you pay:
$$\text{Pay} = 25,000,000 \times 0.0050 = \$125,000$$

**Step 3:** Net roll cash:

$$\text{Net} = \$200,000 - \$125,000 = +\$75,000 \text{ (received)}$$

**Note:** This ignores bid/ask spreads and assumes both trades settle on the same accrual grid. In practice, off-the-run liquidity can be worse, and accrued-premium timing can make realized roll cash differ from this toy calculation.

---

### Example 45.F: Impact of Quoting Convention Error

**Scenario:** A trader mistakenly books a CDX.NA.HY trade using spread convention instead of price convention.

**Given:**
- Notional $N = \$100,000,000$
- Correct quote: Bond price = 102.18%
- Incorrectly assumed: Spread quote treated as coupon + upfront

**What's the error?**

**Correct (price convention):**
$$U_{\%} = 100\% - 102.18\% = -2.18\%$$
$$U_{\$} = 100,000,000 \times 0.0218 = \$2,180,000 \text{ received}$$

If you mistakenly treat the bond price as an upfront quote, you might book +2.18% paid instead of -2.18% received — a $4.36mm cashflow difference on $100mm notional.

**If spread were misinterpreted:**
Suppose the trader thought "102.18" was a spread in bp and applied spread mechanics with a different result — the error could be millions of dollars.

**Lesson:** Always verify the quoting convention for the specific index family before booking. CDX.NA.HY uses bond price; CDX.NA.IG uses spread.

---

### Example 45.G: Index Factor Impact on DV01

**Given:**
- CDX.NA.IG position: $100mm notional, 5Y maturity
- Index DV01 (full factor): $47,000 per bp
- Current factor: $f = 0.976$ (3 defaults have occurred)

**Task:** Calculate actual DV01 exposure.

**Solution:**

$$\text{Actual DV01} = \text{Full DV01} \times f = 47,000 \times 0.976 = \$45,872$$

**Risk implication:** If your risk system shows $47,000 DV01 but the factor is 0.976, your actual exposure is $1,128 less per bp. Over a 10 bp move, that's an $11,280 P&L discrepancy.

---

## 45.7 Practical Notes

### 45.7.1 Booking Checklist for Index Trades

- [ ] **Index identifier:** Family, region, quality, series number
- [ ] **Tenor:** 3Y/5Y/7Y/10Y (note actual maturity is ±3 months from label)
- [ ] **On-the-run vs. off-the-run:** Affects liquidity and roll timing
- [ ] **Quoting convention:** Spread (IG) vs. price (HY/EM)
- [ ] **Fixed coupon:** Series-specific; confirm from confirmation/reference data
- [ ] **Upfront:** Amount and direction (pay/receive)
- [ ] **Settlement date:** Per confirmation/platform; do not assume
- [ ] **Current index factor:** Number of names removed and current factor
- [ ] **Accrued premium:** If trading between coupon dates
- [ ] **Cash reconciliation:** Settlement cash = upfront + accrued premium (check sign and units)

### 45.7.2 Common Pitfalls

1. **Quoting convention confusion:** IG indices quote spread; HY/EM quote price. Mixing them causes large booking errors.

2. **Series/tenor confusion:** "Series 7 5Y" means the 7th vintage of the 5-year contract, not a 7-year contract in Series 5.

3. **Forgetting the ±3 month rule:** A "5Y" index at issuance has ~63 months to maturity, not 60.

4. **Roll date assumptions:** Verify actual roll schedule — market conventions can shift.

5. **Ignoring the index factor:** The index may have fewer than 125 names if defaults have occurred. Notional, DV01, and coupon all scale with the factor.

6. **Regional credit event differences:** CDX excludes restructuring (No-Re); iTraxx includes it (Mod-Re). This affects basis calculations and hedging.

7. **Hedging off-the-run positions:** Hedging an off-the-run series with the on-the-run benchmark can leave residual risk because the portfolios and maturities are not identical.

### 45.7.3 Verification Tests

1. **Quote-object consistency:** \(P_{\text{bond}}=100-U_{\%}\), and for spread-quoted indices \(U_{\$}\approx (S_{I,\text{bp}}-C_{\text{bp}})\text{RPV01}_I\) under the PV01 convention you are using.

2. **Factor bounds:** $f = (M - D)/M$ must be in range $(0, 1]$

3. **Notional consistency:** $N_{\text{out}} = N \times f$

4. **Payment bounds:** Protection payment per default $\leq N/M$

5. **Coupon scaling:** Coupon payments scale linearly with outstanding notional

---

## Summary

- A CDS index identifier pins down the **family/region/quality/series/tenor**; booking the wrong series is booking a different instrument.
- Indices trade with a **fixed running coupon + upfront**; the market quote is either a **spread (bp)** that must be converted using PV01, or a **price per 100** that implies upfront directly.
- Settlement cash at entry is typically **upfront + accrued premium** (clean vs full/dirty). Reconcile these separately before blaming the model.
- After defaults, the **index factor** \(f\) reduces outstanding notional \(N_{\text{out}}=Nf\), which scales premium cashflows and risk.
- With the book’s **down-bump** convention, **CS01** bumps the **pricing spread input** down 1bp; for long protection near par, \(\text{CS01}\approx -\text{RPV01}\).
- Rolls move liquidity from the old to the new series; roll P&L can arise from **composition changes** and **maturity extension** even with unchanged “headline” spread levels.

---

## Key Concepts

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **On-the-run** | Most recently issued series | Concentrates liquidity; benchmark for hedging |
| **Off-the-run** | Previous series (still tradable) | Lower liquidity; may have different constituents |
| **Series** | A specific vintage with fixed coupon and constituent list | Identifies which version you're trading |
| **Index factor ($f$)** | $(M-D)/M$ — fraction of names remaining | Scales notional, DV01, and coupons |
| **Price quote (“bond price”)** | Quote \(P_{\text{bond}}\) per 100; \(U_{\%}=100-P_{\text{bond}}\) | Used in some families to encode upfront directly |
| **Spread quote** | Quote \(S_{I,\text{bp}}\) in bp; \(U_{\$}\approx (S_{I,\text{bp}}-C_{\text{bp}})\text{RPV01}\) | Convert spread → upfront cash under a PV01 convention |
| **Roll** | Transitioning from old to new series | Causes P&L from composition and maturity changes |
| **Notional reduction** | Equal weights: factor drops by \(1/M\) per default (so \(N_{\text{out}}\) drops by \(N/M\)) | Affects future coupon amounts |
| **Flat-curve convention** | Treat quoted index spread as a flat spread curve for PV01 conversion | PV01 (and thus upfront) depends on the chosen convention |
| **No-Re** | No restructuring credit event | Contract mismatch vs Mod-Re / Mod-Mod-Re can create basis |
| **Mod-Mod-Re** | Restructuring included with European-style deliverable limits | Can differ economically from US-style restructuring clauses |
| **Intrinsic spread** | RPV01-weighted average of constituent spreads | Basis = quoted - intrinsic |
| **CS01 (down bump)** | \(PV(\text{spread down }1\text{bp})-PV(\text{base})\) | Requires bump object/size/units; sign differs by long/short protection |

---

## Notation

- \(N\): original index notional (currency).
- \(M\): original number of constituents in the series; \(D\): defaulted names in the series.
- \(f=(M-D)/M\): index factor (unitless); \(N_{\text{out}}=Nf\).
- \(C_{\text{bp}}\): fixed running coupon (bp/year); \(C=C_{\text{bp}}\times 10^{-4}\) in decimal.
- \(S_{I,\text{bp}}\): quoted index spread (bp/year) when spread-quoted; \(S_I=S_{I,\text{bp}}\times 10^{-4}\).
- \(P_{\text{bond}}\): price quote per 100 notional (for price-quoted indices).
- \(U_{\$}\): upfront cash amount (currency), with \(U_{\$}>0\) meaning the **protection buyer pays**; \(U=U_{\$}/N_{\text{out}}\); \(U_{\%}=100U\).
- \(\text{RPV01}_I\): PV of 1 bp/year of running premium for the position (currency per bp). Risky annuity \(A_I := \text{RPV01}_I/(N_{\text{out}}\,10^{-4})\) has units of years.
- \(FP\): auction final price (per 100); \(R=FP/100\); \(LGD=1-R\).
- \(\Delta(t_1,t_2)\): day-count year fraction.
- \(\text{CS01} := PV(\text{spread down }1\text{bp})-PV(\text{base})\) under the stated bump object (currency per bp).

Sign note: in the worked-example cashflow table, negative cashflows are cash **paid by the protection buyer**.

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What does "CDX.NA.IG.42 5Y" mean? | CDX index, North America, Investment Grade, Series 42, 5-year maturity |
| 2 | How many constituents in CDX.NA.IG? | 125 |
| 3 | How many constituents in CDX.NA.HY? | 100 |
| 4 | What is "on-the-run"? | The most recently issued series (concentrates liquidity) |
| 5 | How often do CDS indices roll? | Every six months (March 20 and September 20) |
| 6 | Does a "5Y" index have exactly 5 years to maturity at issue? | No — typically T+3 months at issuance, T-3 months by next roll |
| 7 | How does CDX.NA.IG quote in the market? | By spread (not price) |
| 8 | How does CDX.NA.HY quote in the market? | By bond price (not spread) |
| 9 | Why do HY indices use bond price quoting? | To avoid disputes about index PV01 for spread-to-upfront conversion |
| 10 | Convert bond price 102.18% to upfront | Upfront = 100% - 102.18% = -2.18% (you receive) |
| 11 | Convert bond price 99.50% to upfront | Upfront = 100% - 99.50% = +0.50% (you pay) |
| 12 | Spread-quoted: upfront dollars formula | \(U_{\$}\approx (S_{I,\text{bp}}-C_{\text{bp}})\text{RPV01}\) (under the PV01 convention used) |
| 13 | What is RPV01 (units)? | PV of 1 bp/year of running premium (currency per bp) |
| 14 | What is “points upfront”? | \(U_{\%}=100\,U_{\$}/N_{\text{out}}\) (percent of notional) |
| 15 | Settlement cash at entry (clean vs full) | Full cash ≈ upfront + accrued premium; clean quote focuses on upfront |
| 16 | What happens to index notional when a constituent defaults? | Equal weights: the factor drops by \(1/M\), so \(N_{\text{out}}\) drops by \(N/M\) |
| 17 | Protection payment per default (per name) | \(\frac{N}{M}\left(1-\frac{FP}{100}\right)\) |
| 18 | What credit events trigger CDX vs iTraxx (high level)? | CDX typically excludes restructuring; iTraxx typically includes restructuring (confirm the terms) |
| 19 | Two structural sources of roll P&L | 1) Constituents can change; 2) maturity extends by ~6 months |
| 20 | CS01 definition in this book | \(PV(\text{spread down }1\text{bp})-PV(\text{base})\) for the stated bump object |
| 21 | CS01 sign for long protection (near par, down bump) | Typically negative; \(\text{CS01}\approx -\text{RPV01}\) |
| 22 | First reconciliation step when cash/P&L breaks | Separate upfront vs accrued premium, then check factor and quote object |

---

## Mini Problem Set

1. Parse the identifier "iTraxx Europe Series 21 7Y" and list the components that determine what you are trading.
2. Spread-quoted index: \(N=\$100\)mm, \(f=0.992\), \(C_{\text{bp}}=100\), \(S_{I,\text{bp}}=140\), \(\text{RPV01}=\$42{,}500/\text{bp}\). Compute upfront dollars \(U_{\$}\), points upfront \(U_{\%}\), and pay/receive for a protection buyer.
3. Price-quoted index: \(N=\$50\)mm, \(f=0.95\), \(P_{\text{bond}}=98.25\). Compute upfront dollars and pay/receive for a protection buyer.
4. Factor + coupon: \(M=125\), \(D=3\), \(N=\$200\)mm, \(C_{\text{bp}}=80\), quarterly \(\alpha=0.25\). Compute \(f\), \(N_{\text{out}}\), and the next coupon cashflow.
5. Default settlement: \(N=\$50\)mm, \(M=100\), \(FP=27\). Compute the protection payment (per name and total) and the new outstanding notional after the default.
6. Accrued premium: \(N_{\text{out}}=\$75\)mm, \(C_{\text{bp}}=100\), ACT/360, 37 days since last coupon. Compute accrued premium.
7. Roll cash: roll \(N=\$30\)mm from off-the-run (receive 0.45% to close) to on-the-run (pay 0.20% to enter). Compute net roll cash.
8. CS01: a long-protection index position has \(\text{RPV01}=\$18{,}200/\text{bp}\). Using the down-bump definition, approximate CS01 and interpret the sign.
9. Concept: explain two structural sources of roll P&L even if spreads did not “move” on the screen.
10. Desk/ops: system A books only upfront; system B reports settlement cash. What two cash components should you reconcile first?

### Solution Sketches (Selected)

**2.** \(N_{\text{out}}=99.2\)mm. Spread difference \(=40\) bp. \(U_{\$}\approx 40\times 42{,}500=\$1{,}700{,}000\) paid; \(U_{\%}\approx 100\times 1.7/99.2=1.71\%\) paid.

**3.** \(U_{\%}=100-98.25=1.75\%\) paid. \(N_{\text{out}}=47.5\)mm, so \(U_{\$}=47.5\text{mm}\times 0.0175=\$831{,}250\) paid.

**8.** \(\text{CS01}\approx -\text{RPV01}=-\$18{,}200/\text{bp}\) for long protection (spreads down reduces PV).

**9.** (i) The constituent portfolio can change between series; (ii) the new on-the-run contract is typically ~6 months longer maturity, changing PV01/fair spread even with identical names.

**10.** Reconcile (i) upfront and (ii) accrued premium (then check factor and quote object).

---

## References

- O'Kane, *Modelling Single-name and Multi-name Credit Derivatives* (CDS portfolio indices: index families/counts, quote objects, roll/maturity grid, default handling, restructuring clauses and basis)
- Hull, *Options, Futures, and Other Derivatives* (credit indices overview; update/roll timing; pricing intuition for fixed coupon + upfront)
- Hull, *Risk Management and Financial Institutions* (credit indices; fixed coupons and upfront quoting regime)
- Neftci, *Principles of Financial Engineering* (roll mechanics; points-upfront / fixed-coupon regime intuition)
- Pachamanova and Fabozzi, *Simulation and Optimization in Finance: Modeling with MATLAB, @RISK, or VBA* (CDX mechanics; equal weights; notional reduction after defaults)
