# Chapter 10: Treasury Market Microstructure for Relative Value

---

## Fact Classification

### (A) Verified Facts (Source-Backed)

- Treasuries are issued via auction; institutions can submit competitive bids (quantity+yield) and noncompetitive bids (quantity only), with noncompetitive filled at the average yield of accepted competitive bids
- The on-the-run Treasury is the most recently issued security of a given maturity; older issues are off-the-run and generally less liquid with wider bid-ask spreads
- Repo is economically like a collateralized loan implemented as a sale and repurchase: the repurchase price equals principal times $(1 + rd/360)$ in the book's simple-interest convention
- In the repo market, general collateral (GC) refers to borrowing/lending cash against a broad class of acceptable collateral; special collateral refers to a repo against a particular security, usually at a lower repo rate when that security is in demand ("special")
- The book defines the special spread as $r_{GC} - r_{sp}$. Special repo rates are bounded below (effectively by 0 in the book's discussion) because a trader unable to borrow to make delivery can fail rather than accept a negative special rate; therefore, special spreads cannot exceed $r_{GC}$
- On-the-run Treasuries often exhibit a liquidity premium, with two components emphasized: (i) a financing advantage (ability to borrow at below-GC rates using the security as collateral), and (ii) a pure liquidity premium (value of holding the most liquid benchmark)
- The book describes an auction-cycle pattern for special spreads: spreads are small right after auctions and tend to rise, often peaking before the next auction; reopenings behave differently from new issues in the pattern shown
- Using on-the-run yields as "fundamental" curve points or benchmarks can be misleading because OTR yields embed liquidity premiums and special financing effects; one remedy is to fit a curve using off-the-run data or adjust OTR yields for liquidity/financing effects
- The book shows how to translate a financing advantage into a yield-equivalent by dividing (price-point financing advantage) by $DV01$
- Long/short positions and repo mechanics: to short a bond, a desk can use a reverse repo (lending cash to borrow the bond), and if it cannot cover, it must "roll" the short by extending repo or borrowing elsewhere

### (B) Reasoned Inference (Derived from A)

- "Relative value" in Treasuries is not only about term-structure expectations; it is also about microstructure wedges: liquidity and financing create persistent deviations between otherwise similar cash flows
- A clean decomposition for many OTR/off-run RV questions is:

$$\text{observed yield/price difference} \approx \underbrace{\text{cash-flow/curve component}}_{\text{macro}} + \underbrace{\text{financing component}}_{\text{repo specialness}} + \underbrace{\text{pure liquidity component}}_{\text{benchmark demand}}$$

  Only the first is "fundamental curve"; the latter two are microstructure.

- Any hedge that uses an OTR benchmark inherits exposure to liquidity/specialness shocks, even if macro rates are unchanged

### (C) Speculation (Clearly Labeled; Minimal)

- **I'm not sure** about the exact current U.S. Treasury announcement/auction/issue/settlement timeline (dates and sequencing) from these sources alone. To be certain, we would need the precise U.S. Treasury auction rules (or a specific book section that enumerates the calendar).
- **I'm not sure** about modern "fails charges," fails netting mechanics, or post-crisis institutional rules beyond what is stated (e.g., the special-rate floor logic). To be certain, we would need a source section explicitly detailing fails treatment and penalties.

---

## Conventions & Notation

### Default Market Context

We treat U.S. Treasury trading as an OTC dealer market, with auctions feeding the secondary market and repo financing supporting inventories and shorts. The core microstructure mechanisms emphasized are: on-the-run (OTR) liquidity, repo specialness, and auction/issuance cycles.

### Interest and Accrual Conventions

Repo interest is computed with simple-interest conventions shown in the sources (360 denominator). Coupon accrual conventions can differ in practice; where we need accrual numbers for examples, we state toy assumptions.

### Notation Glossary

| Symbol | Definition |
|--------|------------|
| $P$ | Clean price per 100 |
| $AI$ | Accrued interest per 100 |
| $y$ | Yield-to-maturity (decimal) |
| $\Delta y_{\text{bp}}$ | Yield change in basis points |
| $DV01$ | Price change per 100 for a 1 bp yield change |
| $r_{GC}$ | General collateral repo rate |
| $r_{sp}$ | Special repo rate |
| $s = r_{GC} - r_{sp}$ | Special spread |
| $d$ | Day count (days) |
| $P_{fwd}$ | Forward clean price |
| $F$ | Futures price |
| $cf$ | Conversion factor |

---

## Core Concepts

### 1) Treasury Trading Venue and Quoting

**Formal Definition:**

The Treasury market is described as an over-the-counter (OTC) market where dealers/financial institutions make markets; trading is not centralized like a pure exchange.

Treasury prices can be quoted in fractions of par (e.g., 1/32 increments) and bid-ask spreads vary by liquidity.

**Intuition:**

"Microstructure" matters because the traded price/yield includes transaction costs, inventory constraints, and liquidity preference.

**Trading / Risk / Portfolio Practice:**

- Bid-ask and market impact set a floor on exploitable RV (expected microstructure "alpha" must exceed trading costs)
- Liquidity differences can dominate short-horizon P&L even when "rates" are stable

---

### 2) On-the-Run vs Off-the-Run

**Formal Definition:**

- **On-the-run (OTR):** The most recently issued Treasury security of a given maturity sector
- **Off-the-run:** Older issues

**Intuition:**

OTR issues concentrate trading, hedging, and benchmark demand. Off-the-runs may be "buy-and-hold" and trade less.

**Trading / Risk / Portfolio Practice:**

- OTR trades often have tighter bid-ask and are preferred hedges
- OTR yields may be lower (prices higher) than near-maturity off-the-runs due to a liquidity premium (defined below)

---

### 3) Repurchase Agreements (Repo)

**Formal Definition:**

A repo is a transaction where one party buys securities and agrees to sell them back later at a higher price; the implied interest is the repo rate.

**Example:** Repurchase price $= 100{,}000{,}000 \times (1 + 0.0545/360)$ for overnight at 5.45%.

**Intuition:**

Repo is the financing "plumbing" that allows dealers to hold inventory and allows shorts to source bonds.

**Trading / Risk / Portfolio Practice:**

- Carry in Treasury RV is largely "coupon minus repo"
- Repo haircuts/repricing provisions exist (collateral > cash lent; margining), though the book abstracts from them in many calculations

---

### 4) General Collateral (GC) vs Special Repo

**Formal Definition:**

- **GC repo:** Cash borrowed/lent against collateral drawn from a broad acceptable set (e.g., "any Treasury"), with rate $r_{GC}$
- **Special repo:** Repo against a specific security; rate $r_{sp}$ is usually below GC when that security is in high demand to borrow
- **Special spread:** $s = r_{GC} - r_{sp}$

**Intuition:**

"Special" means the market is paying (via accepting a low repo rate) to obtain a particular bond.

**Trading / Risk / Portfolio Practice:**

- Specialness changes the financing cost of being long and the borrow cost of being short
- Term repos exist to lock in rates and reduce overnight risks tied to both money rates and collateral supply/demand shocks

---

### 5) Liquidity Premium

**Formal Definition:**

In the book's framing, the OTR liquidity premium has two components:

1. **Financing advantage:** Ability to borrow money at below-GC rates when using the security as collateral
2. **Pure liquidity premium:** Value investors place on holding the most liquid benchmark, independent of financing

**Intuition:**

OTR securities are "special" in both trading liquidity and repo funding.

**Trading / Risk / Portfolio Practice:**

RV desks track OTR/off-run yield gaps as a proxy for liquidity/financing conditions, not as a stable "fair value constant."

---

### 6) Financing Advantage via Repo

**Formal Definition:**

The financing advantage is the benefit from financing at $r_{sp}$ rather than $r_{GC}$, applied to the funded amount. The book converts this to yield-equivalent terms by dividing by $DV01$.

**Intuition:**

If you can fund a position cheaper, you can rationally pay a higher price (accept a lower yield).

**Trading / Risk / Portfolio Practice:**

Funding differences can explain why a bond looks "rich" on a spot-yield basis but is fair once carry/financing is included.

---

### 7) Auction Cycle, Issuance, and "When-Issued" Trading

**Formal Definition:**

Auctions can be new issues (creating a new on-the-run) or reopenings (adding supply to an existing on-the-run); the special spread pattern in the book is marked by auction dates (vertical lines).

The sources indicate that new issues can trade when-issued before they become the new OTR (e.g., "When-issued 10-year..." in a data table; and a narrative example of a new five-year trading when-issued).

**Intuition:**

Issuance changes the available float, affecting both liquidity and specialness.

**Trading / Risk / Portfolio Practice:**

Auction dates matter for:
- OTR/off-run "roll" dynamics
- Repo specialness paths
- Temporary cheapening/richening due to inventory positioning (conceptually consistent with dealer inventory and seasoning; secondary source notes dealers may hold new issues until "seasoned" and distributed)

---

### 8) Shorting Constraints, Fails, and the Special-Rate Floor

**Formal Definition:**

If a trader cannot borrow a security to make delivery, it may fail, losing one day's interest on the sale proceeds; this makes failing economically similar to borrowing at a 0% repo rate, supporting a floor at 0 and implying the special spread cannot exceed $r_{GC}$.

**Intuition:**

When a bond is hard to borrow, the short's cost rises until it hits a boundary where failing dominates.

**Trading / Risk / Portfolio Practice:**

"Short squeeze" type dynamics manifest as widening special spreads and distorted relative value in cash yields.

---

### 9) Benchmark Distortion and Curve/Spread Measurement

**Formal Definition:**

OTR securities embed liquidity premiums and special financing; therefore, constructing a term structure from OTR yields can be problematic. The book suggests alternatives like discarding OTRs, weighting them differently, or adjusting yields for liquidity/financing effects.

Similarly, using OTR yields in swap spread measurement can be misleading; alternatives include fitted curves or adjusting OTR yields for liquidity premiums.

**Intuition:**

A "benchmark yield" can move for microstructure reasons, not purely because discount rates changed.

**Trading / Risk / Portfolio Practice:**

RV signals that use OTR benchmarks (swap spreads, asset swap spreads, curve point spreads) can be biased.

---

## Math and Derivations

### 1) Price-Yield Linearization and DV01

**Derivation:**

By definition, yield-based DV01 is the price change for a 1 bp increase in yield:

$$\boxed{DV01 \equiv -\frac{\partial P}{\partial y} \times 0.0001}$$

In linear approximation:

$$\boxed{\Delta P \approx -DV01 \cdot \Delta y_{\text{bp}}}$$

The book provides a practical expression:

$$DV01 \approx \frac{P \cdot D_{mod}}{10000}$$

where $D_{mod}$ is modified duration and $P$ is price per 100.

**Unit Checks and Sanity Checks:**

- $DV01$: "price points per bp" (e.g., 0.085 means 0.085 points per bp)
- If $\Delta y > 0$ (yields up), $\Delta P < 0$ for standard bonds

---

### 2) Repo Interest and Financing Cost

**Derivation:**

A repo over $d$ days with principal (cash lent/borrowed) $X$ and rate $r$ has repurchase price:

$$\boxed{X_{rep} = X\left(1 + \frac{rd}{360}\right)}$$

consistent with the repo examples in the book.

For a long bond position financed in repo, approximate financing cost over $d$ days (per $100 par) is:

$$\text{FinCost} \approx (P + AI) \cdot \frac{rd}{360}$$

**Unit Checks and Sanity Checks:**

- $r$ is annualized; multiplying by $d/360$ gives a period fraction
- A higher repo rate raises the cost of funding a long position

---

### 3) Carry (Interest Income Minus Financing Cost) and P&L Decomposition

**Derivation:**

The book decomposes P&L over a holding period as:

$$P\&L = \Delta(P + AI) + \text{Carry}$$

where carry equals interest income minus financing cost.

It gives a practical "carry-only" expression (for unchanged prices) as:

$$\boxed{\text{Carry} = \left[(AI_1 - AI_0) + \text{Coupon received}\right] - (P + AI_0) \cdot r \cdot \frac{d}{360}}$$

with the idea that the relevant funded amount is the invoice price and the financing cost uses money-market day count conventions.

**Unit Checks and Sanity Checks:**

- Carry in price points: compare to bid-ask in price points (e.g., 1/32 = 0.03125 points)
- In "normal" upward-sloping curves, coupon may exceed repo, producing positive carry (book discussion in forward pricing)

---

### 4) Special Spread and Financing Advantage

**Derivation:**

Special spread (definition):

$$\boxed{s \equiv r_{GC} - r_{sp}}$$

Financing advantage over $d$ days (per $100 par), approximating funded amount by invoice price:

$$\boxed{\text{FinAdv}_{100} \approx (P + AI) \cdot s \cdot \frac{d}{360}}$$

Yield-equivalent financing advantage:

$$\boxed{\text{FinAdv}_{\text{bp}} \approx \frac{\text{FinAdv}_{100}}{DV01}}$$

This is exactly the conversion used in the book's example: financing advantage divided by DV01 gives "basis points in yield terms."

**Unit Checks and Sanity Checks:**

- $\text{FinAdv}_{100}$ is in price points; dividing by $DV01$ (price points per bp) yields bp
- If $r_{sp} < r_{GC}$, then $s > 0$: a long funded special has lower funding cost, i.e., positive advantage

---

### 5) Forward Price from Repo (Microstructure Enters Through Repo)

**Derivation:**

For a coupon bond with no coupon payments during the forward window, the book derives:

$$\boxed{P_{fwd} + AI(d) = (P(0) + AI(0))\left(1 + \frac{rd}{360}\right)} \quad \text{(16.7)}$$

Rearranged:

$$\boxed{P_{fwd} = P(0) - \text{Carry}} \quad \text{(16.8)}$$

where:

$$P_{fwd} = P(0) - \left(AI(d) - AI(0) - (P(0) + AI(0))\frac{rd}{360}\right)$$

**Sanity Checks:**

If carry is positive (income > financing cost), the forward price is lower than spot, consistent with the book's interpretation.

---

### 6) Futures Basis (Preview)

**Derivation:**

For a deliverable bond $i$, the book defines:

$$\boxed{GB_i(t) = P_i(t) - cf_i \cdot F(t)} \quad \text{(20.10)}$$

$$\boxed{NB_i(t) = P_{fwd}^i(t) - cf_i \cdot F(t)} \quad \text{(20.11)}$$

Using $P_{fwd} = P - \text{carry}$, net basis becomes:

$$\boxed{NB_i(t) = GB_i(t) - \text{carry}_i(t)} \quad \text{(20.12)}$$

**Assumptions (Explicit):**

This chapter does not develop CTD, delivery options, or full Treasury futures microstructure; we only use these definitions as a bridge to "implied repo" arithmetic.

---

## Measurement & Risk

### How Auction Cycles, Liquidity, and Financing Show Up In:

#### Observed Yields/Spreads

- **OTR vs off-run yields:** OTR issues can trade at lower yields due to liquidity premium, which includes both financing advantage and pure liquidity demand
- **Special spreads:** Specialness varies over time and exhibits an auction-cycle pattern: small after auctions, rising toward the next auction, with differences for reopenings vs new issues
- **Spot vs forward yield spreads:** Even if the spot yield spread between two issues is small, expected differences in future specialness can create significant differences in forward yields/spreads (book example logic)

#### Carry/Rolldown

- Carry explicitly depends on the repo financing rate in the book's decomposition; if a bond goes special (repo rate falls), carry improves for a levered long funded in repo
- Rolldown signals can be distorted if the curve "point" used for comparison is an OTR yield that embeds a time-varying liquidity premium; the book notes curve fitting is complicated by OTR liquidity premium and special financing effects

#### Relative Value Signals

- A "cheap/rich" signal based on observed yields may flip after adjusting for financing advantage; the book converts financing advantage to yield-equivalent by dividing by DV01, enabling apples-to-apples comparison across issues
- Auction-cycle information matters because special spreads are not constant; expected changes in specialness affect forward valuation and RV decisions

#### Hedging Outcomes

If you hedge with an OTR benchmark, you are exposed to changes in its liquidity premium and specialness. The book explicitly warns that using OTR yields for swap spreads and term-structure extraction can be misleading and suggests alternatives (fitted curves, adjustments).

---

### Key Definitions and Interpretations

#### "Liquidity Premium" (Observed Price/Yield Difference, Not a Fundamental Constant)

**Definition (operational):** The part of the observed OTR-off-run price/yield difference that cannot be explained by cash-flow differences under a smooth curve, and is attributed to (i) financing advantage and (ii) pure liquidity in the book's framing.

**Interpretation:** It is time-varying, sensitive to market stress, dealer balance sheet, and benchmark demand. (Time-varying is a logical implication; the book's empirical patterns in special spreads and curve-fitting complications support non-constancy.)

#### "Financing Advantage" via Repo (GC vs Special)

**Definition:** The benefit of being able to finance (or equivalently borrow cash against) a bond at a rate below GC because that bond is special collateral.

**Computation:** $\text{FinAdv}_{100} \approx (P + AI) \cdot (r_{GC} - r_{sp}) \cdot d/360$, and yield-equivalent $\approx \text{FinAdv}_{100}/DV01$.

#### Link Between Specialness and Shorting Constraints

**Mechanism:** Shorts need to borrow the specific bond to deliver; the desire to borrow drives specialness (low $r_{sp}$). When borrowing becomes too costly (would require negative special rates), participants may fail, implying a floor at 0% and bounding the special spread by $r_{GC}$.

---

## Worked Examples

*All examples use toy numbers but realistic magnitudes (bp, price per 100, days). We explicitly state simplifying assumptions.*

---

### Example A: On-the-Run Premium in Yield Terms

**Objective:** Two Treasuries with similar maturity—compute yield difference and translate into a price difference using DV01. Interpret as a "liquidity premium" proxy.

**Given (toy but realistic):**

- OTR 5-year: $y_{OTR} = 4.80\%$
- Off-run 5-year: $y_{off} = 4.83\%$
- Yield difference: $\Delta y = y_{off} - y_{OTR} = 0.03\% = 3$ bp
- $DV01 = 0.045$ price points per bp (per $100 par)
- Notional: $100,000,000 face

**Step 1: Convert yield difference to price difference (per $100 par)**

$$\Delta P \approx -DV01 \cdot \Delta y_{\text{bp}}$$

Off-run has higher yield, so OTR is richer (higher price). Magnitude:

$$|\Delta P| \approx 0.045 \times 3 = 0.135 \text{ points per 100}$$

**Step 2: Convert price points to dollars**

$100mm: 1.00 price point = $1,000,000

$$\$\Delta \approx 0.135 \times \$1{,}000{,}000 = \$135{,}000$$

**Interpretation (microstructure lens):**

A 3 bp yield "richness" corresponds to $135k per $100mm—large relative to many transaction-cost budgets. Per the book, such OTR richness can proxy a liquidity premium, which itself includes financing advantage and pure liquidity value rather than a fundamental curve difference.

---

### Example B: Auction Cycle Stylized Timeline

**Objective:** Construct a 2-3 week timeline around an auction and show a toy series of (i) on-the-run yield vs off-the-run yield and (ii) specialness (GC repo vs special repo). Compute the implied $ financing benefit/cost over the period.

**Source-backed anchor for the pattern:** Special spreads are small just after auctions and tend to rise toward the next auction; reopenings differ from new issues.

**Toy Setup:**

- Horizon: 15 days (days 0-14), where day 0 is an auction/issue event (stylized)
- Notional: $100,000,000 financed in repo
- Assume invoice price $\approx \$100$ per $100 par for funding amount (simplifying)
- $r_{GC} = 5.00\%$ constant (toy)
- Special repo rate drifts down as specialness rises (toy)

**Toy Time Series (selected columns):**

| Day | $y_{OTR}$ (%) | $y_{off}$ (%) | $r_{GC}$ (%) | $r_{sp}$ (%) | Special spread $s$ (bp) |
|-----|---------------|---------------|--------------|--------------|-------------------------|
| 0   | 4.860         | 4.850         | 5.00         | 4.90         | 10                      |
| 4   | 4.840         | 4.850         | 5.00         | 4.00         | 100                     |
| 9   | 4.828         | 4.850         | 5.00         | 2.50         | 250                     |
| 14  | 4.822         | 4.850         | 5.00         | 2.00         | 300                     |

(Daily special spreads used in the sum: 10, 20, 40, 70, 100, 120, 150, 190, 220, 250, 270, 280, 290, 295, 300.)

**Step 1: Compute total financing benefit from specialness**

Financing benefit (vs GC) over one day:

$$\text{Benefit}_{day} = N \cdot \frac{s}{10000} \cdot \frac{1}{360}$$

Over 15 days:

$$\text{Benefit}_{15d} = N \cdot \frac{1}{360} \cdot \sum_{k=0}^{14} \frac{s_k}{10000}$$

Sum of daily spreads:

$$\sum s_k = 2605 \text{ bp-days}$$

So:

$$\text{Benefit}_{15d} = 100{,}000{,}000 \times \frac{1}{360} \times \frac{2605}{10000}$$

Compute the fraction:

$$\frac{2605}{10000 \times 360} = \frac{2605}{3{,}600{,}000} \approx 0.000723611$$

Therefore:

$$\text{Benefit}_{15d} \approx 100{,}000{,}000 \times 0.000723611 = \$72{,}361$$

**Step 2 (optional): Translate financing benefit to yield-equivalent**

Assume (toy) $DV01 = 0.085$ points per bp (per $100 par) for a 10-year-like security.

DV01 in dollars for $100mm: $0.085$ points per bp $\Rightarrow \$85{,}000$ per bp.

So yield-equivalent of $72,361:

$$\Delta y_{eq} \approx \frac{72{,}361}{85{,}000} \approx 0.85 \text{ bp}$$

This mirrors the book's conversion logic (price-point advantage divided by DV01 gives bp).

**Interpretation:**

Over just ~3 weeks, specialness can contribute meaningful carry ($72k on $100mm) even with no macro rate move. The OTR yield in the toy also richens vs off-run (OTR yield falls), consistent with the idea that OTR liquidity premium can build over time (conceptual; the book explicitly documents the special-spread pattern and liquidity-premium components).

---

### Example C: Financing Advantage Math

**Objective:** Given a bond position and two repo rates (GC and special), compute the difference in financing cost over $N$ days. Translate to bp of yield-equivalent advantage using DV01.

**Given:**

- Notional: $100,000,000 face
- Clean price $P = 101.50$, accrued $AI = 0.80$
- Invoice per 100: $P + AI = 102.30$
- Horizon $d = 30$ days
- $r_{GC} = 5.10\%$, $r_{sp} = 3.60\%$
- $DV01 = 0.0435$ points per bp (per $100) (similar scale to the book's 5-year example)

**Step 1: Financing cost per $100 at GC**

$$\text{FinCost}_{GC,100} = (P + AI) \cdot r_{GC} \cdot \frac{d}{360}$$

Compute $r_{GC} \cdot d/360 = 0.0510 \cdot 30/360 = 0.0510/12 = 0.00425$.

$$\text{FinCost}_{GC,100} = 102.30 \times 0.00425 = 0.434775$$

**Step 2: Financing cost per $100 at special**

$$\text{FinCost}_{sp,100} = 102.30 \cdot 0.0360 \cdot \frac{30}{360}$$

Compute $0.0360 \cdot 30/360 = 0.0360/12 = 0.003$.

$$\text{FinCost}_{sp,100} = 102.30 \times 0.003 = 0.3069$$

**Step 3: Financing advantage per $100**

$$\text{FinAdv}_{100} = 0.434775 - 0.3069 = 0.127875 \text{ points}$$

**Step 4: Dollar financing advantage on $100mm**

1 price point on $100mm = $1,000,000

$$\text{FinAdv}_{\$} = 0.127875 \times 1{,}000{,}000 = \$127{,}875$$

**Step 5: Yield-equivalent advantage**

$$\text{FinAdv}_{\text{bp}} \approx \frac{0.127875}{0.0435} \approx 2.94 \text{ bp}$$

This is exactly the book's method: convert financing advantage to yield-equivalent by dividing by $DV01$.

---

### Example D: Shorting Constraint Economics

**Objective:** Show the economics of being short a "special" Treasury: compute the incremental cost of borrowing at a special rate vs GC.

**Source-backed bounds:** The book argues special repo rates are bounded below (effectively at 0) because failing becomes preferable to paying a negative special rate; thus special spread $\leq r_{GC}$.

**Toy Setup:**

- You are short $100,000,000 face of a particular OTR note
- To maintain the short, you must borrow the security; the book describes doing this via reverse repo (lending cash motivated by the need to borrow particular bonds)
- GC rate $r_{GC} = 5.00\%$
- Special rate $r_{sp} = 0.50\%$ (highly special)
- Horizon $d = 7$ days
- Approximate cash posted $\approx \$100mm$ (toy simplification)

**Step 1: Incremental cost vs GC**

As the cash lender in reverse repo, you earn $r_{sp}$ instead of $r_{GC}$. Opportunity cost:

$$\Delta\text{Cost} = N \cdot (r_{GC} - r_{sp}) \cdot \frac{d}{360}$$

Compute spread:

$$r_{GC} - r_{sp} = 0.0500 - 0.0050 = 0.0450$$

Compute:

$$\Delta\text{Cost} = 100{,}000{,}000 \times 0.0450 \times \frac{7}{360}$$

First $0.0450 \times 7 = 0.3150$. Then $0.3150/360 = 0.000875$.

$$\Delta\text{Cost} = 100{,}000{,}000 \times 0.000875 = \$87{,}500$$

**Step 2: Relate to the "fails / floor" logic**

If specialness tried to push $r_{sp}$ below 0, the book's logic says failing dominates (economically similar to a 0% special), bounding the special rate and capping the special spread at $r_{GC}$.

**Interpretation:**

Even without any yield move, being short a very special issue can be expensive in carry terms ($87.5k/week on $100mm), explaining why "hard-to-borrow" securities can look rich in cash yields.

---

### Example E: When-Issued / Auction Supply Effect (Conceptual with Numbers)

**Objective:** Use a toy supply shock around issuance to illustrate how yields can temporarily cheapen/richen. Compute the implied RV trade P&L under stated assumptions.

**What we can source vs what we can't:**

*Verified from sources:*
- New issues can trade when-issued before they become the new on-the-run (table and narrative)
- Auction-cycle dynamics affect specialness in systematic ways

*I'm not sure* (from these sources alone) about the precise, empirically correct sign/magnitude of "cheapening into auction" for yields in today's market. To be certain, we would need either (i) a specific empirical section in the sources quantifying this, or (ii) an explicitly cited market-microstructure study.

**Toy Scenario (explicitly stylized):**

- A new 10-year note trades when-issued
- Assume into the auction the WI yield cheapens by +5 bp due to supply/inventory positioning; after issuance it richens by -4 bp as benchmark demand develops
- DV01 (10-year-like): $DV01 = 0.085$ points per bp (per $100)
- Notional: $100,000,000

**Toy yield path:**

- At auction (enter trade): $y = 4.95\%$
- Five days later (exit): $y = 4.91\%$
- Yield change: $\Delta y = -4$ bp

**Step 1: Unhedged price P&L**

$$\Delta P \approx -DV01 \cdot \Delta y_{\text{bp}} = -0.085 \cdot (-4) = +0.340 \text{ points}$$

Dollar:

$$\$P\&L \approx 0.340 \times \$1{,}000{,}000 = \$340{,}000$$

**Step 2: DV01-matched hedge to isolate relative/microstructure move**

Suppose a DV01-matched hedge (e.g., an off-the-run 10-year proxy) rallied by 3 bp over the same window due to macro rates:

- Hedge yield change: $-3$ bp
- Relative move of WI vs hedge: $(-4) - (-3) = -1$ bp (WI richens 1 bp more)

Residual (relative) P&L:

$$\Delta P_{rel} \approx -DV01 \cdot (-1) = +0.085 \text{ points} \Rightarrow \$85{,}000$$

**Interpretation:**

Even if most of the yield move is "macro," the extra 1 bp richening can be thought of as an issuance/microstructure effect (toy), and it is economically meaningful.

---

### Example F: Benchmark Distortion

**Objective:** Show how using an on-the-run yield as a "curve point" can bias a spread measurement versus using a fitted/smoothed curve. Provide a toy bond spread calculation under both benchmarks and compare.

**Source-backed principle:** OTR yields embed liquidity premiums and special financing, so using them as benchmarks (e.g., swap spreads) can be misleading; using fitted curves or adjustments is proposed.

**Toy Setup:**

- Consider a 5-year swap fixed rate: $R_{swap} = 5.20\%$ (toy)
- OTR 5-year Treasury yield: $y_{OTR} = 4.80\%$ (toy)
- Fitted (off-the-run-based) 5-year curve yield: $y_{fit} = 4.85\%$ (toy; 5 bp higher than OTR)

**Step 1: Swap spread using OTR**

$$\text{SwapSpread}_{OTR} = R_{swap} - y_{OTR} = 5.20\% - 4.80\% = 0.40\% = 40 \text{ bp}$$

**Step 2: Swap spread using fitted curve**

$$\text{SwapSpread}_{fit} = R_{swap} - y_{fit} = 5.20\% - 4.85\% = 0.35\% = 35 \text{ bp}$$

**Step 3: Distortion**

$$\Delta = 40 - 35 = 5 \text{ bp}$$

**Interpretation:**

A 5 bp difference is large relative to typical bid-ask in many rates products, and it is purely a benchmark choice issue. This is exactly why the book cautions against taking OTR yields as "fundamental" curve points.

---

### Example G: Link to Futures Basis (Optional Preview)

**Objective:** Provide a simple "implied repo" style calculation linking cash price, futures price, and carry over the delivery window.

**What is source-backed:**

- Forward price from repo (no coupon in window): $P_{fwd} + AI(d) = (P(0) + AI(0))(1 + rd/360)$
- Basis and carry relations: $GB = P - cf \cdot F$, $NB = P_{fwd} - cf \cdot F = GB - \text{carry}$

**What is not fully developed here:**

*I'm not sure* about CTD selection, delivery options, invoice-price conventions in full detail from the limited excerpts used here. To be certain, we'd need the specific Treasury futures delivery mechanics section (not just the basis definitions) and/or the full CTD discussion.

**Toy implied repo calculation (clean preview, no coupon paid during window):**

**Given:**

- Spot clean price $P(0) = 100.50$
- Accrued interest today $AI(0) = 0.80$
- Days to delivery $d = 60$
- Accrued interest at delivery $AI(d) = 1.20$ (toy)
- Futures price $F = 112.00$
- Conversion factor $cf = 0.90$

**Approximate futures-implied forward clean price:**

$$P_{fwd} \approx cf \cdot F = 0.90 \times 112.00 = 100.80$$

(This is a simplifying assumption for preview purposes.)

**Step 1: Solve for implied repo rate $r$ using (16.7)**

$$P_{fwd} + AI(d) = (P(0) + AI(0))\left(1 + \frac{rd}{360}\right)$$

Compute left side:

$$P_{fwd} + AI(d) = 100.80 + 1.20 = 102.00$$

Compute right-side base:

$$P(0) + AI(0) = 100.50 + 0.80 = 101.30$$

Take ratio:

$$\frac{102.00}{101.30} \approx 1.006909$$

So:

$$1 + \frac{rd}{360} = 1.006909 \Rightarrow \frac{rd}{360} = 0.006909$$

With $d = 60$:

$$r = \frac{360}{60} \times 0.006909 = 6 \times 0.006909 = 0.041454 \approx 4.15\%$$

**Step 2: Compute carry implied by this repo**

From (16.8), $P_{fwd} = P(0) - \text{Carry}$. Rearranged:

$$\text{Carry} = P(0) - P_{fwd} = 100.50 - 100.80 = -0.30 \text{ points}$$

Alternatively, using the "income minus financing" intuition:

- Interest income from accrual: $AI(d) - AI(0) = 1.20 - 0.80 = 0.40$
- Financing cost:

$$(P(0) + AI(0)) \cdot r \cdot \frac{d}{360} = 101.30 \times 0.041454 \times \frac{60}{360}$$

Since $0.041454 \times 60/360 = 0.006909$,

$$\text{FinCost} = 101.30 \times 0.006909 \approx 0.700 \text{ points}$$

Carry $= 0.40 - 0.700 = -0.300$ points (matches).

**Step 3: Basis check**

$$GB = P - cf \cdot F = 100.50 - 100.80 = -0.30$$

$$NB = GB - \text{carry} = -0.30 - (-0.30) = 0$$

This aligns with the identity $NB = GB - \text{carry}$.

---

## Practical Notes

### Market Mechanics Vocabulary

| Term | Definition |
|------|------------|
| **On-the-run vs off-the-run** | OTR is the most recently issued Treasury of a maturity; off-the-runs are older and typically less liquid |
| **Auction cycle** | Auctions introduce new supply (new issue) or add to supply (reopening). The book's empirical discussion ties auction dates to systematic patterns in special spreads and, by implication, financing conditions. When-issued trading exists before the new issue becomes the new OTR |
| **Specials vs GC** | GC repo: broad collateral class; special repo: specific security; special rates often lower than GC when that security is in demand |
| **Liquidity and "flight to quality" effects** | The provided general derivatives/risk texts describe "flight to quality" episodes (e.g., widening credit spreads, liquidity stress) and note how liquidity can evaporate in crises. This is relevant as context for why liquidity premiums can spike |

### Common Pitfalls

1. **Treating on-the-run yields as "fundamental curve points"** without adjusting for liquidity premium and special financing effects (the book explicitly flags this)

2. **Ignoring financing when evaluating RV:** Repo specialness can dominate carry and forward valuation

3. **Comparing securities with different liquidity/financing characteristics "as if identical":** OTR vs off-run can differ in both pure liquidity and financing advantage

### Implementation Pitfalls

1. **Inconsistent clean vs dirty handling:** Financing uses the funded invoice amount, and the book's forward-price derivation explicitly uses $P + AI$

2. **Day count mismatch:** Repo interest uses 360 denominator in the book; coupon accrual can differ and must be handled consistently

3. **Sign mistakes:**
   - Long financed in repo: higher repo rate $\rightarrow$ higher cost
   - Short requires borrowing security via reverse repo: a lower special rate means lower interest earned on cash posted $\rightarrow$ higher cost (lost carry)

### Verification Tests

1. **DV01 translation:** Confirm $\$P\&L \approx DV01_{\$} \times \Delta y_{\text{bp}}$ and $\Delta P_{100} \approx DV01 \times \Delta y_{\text{bp}}$

2. **Sanity-check magnitudes:**
   - Compare liquidity premium proxies (bp) to bid-ask (in 32nds) and to financing carry over realistic horizons
   - The book gives a transaction-cost benchmark: $1/32\%$ on $100mm is $31,250, illustrating that microstructure costs matter relative to small RV edges

3. **Financing:** Confirm directionality—specialness ($r_{sp} < r_{GC}$) increases carry for levered longs

---

## Summary & Recall

### 10-Bullet Executive Summary

1. Treasuries trade in an OTC dealer market; microstructure (liquidity + financing) matters for RV
2. On-the-run issues are the most recent and tend to be more liquid than off-the-runs
3. Repo is the key financing mechanism; repo math uses simple interest with a 360 denominator in the book's examples
4. Special repo means a specific bond is demanded; it borrows/lends at a lower rate than GC
5. Special spread $= r_{GC} - r_{sp}$ is a central state variable for Treasury RV
6. OTR liquidity premium has two components: financing advantage + pure liquidity value
7. Special spreads follow an auction-cycle pattern in the book: small after auctions, rising toward the next auction
8. Shorting constraints and failing logic bound specialness (special rates effectively not negative in the book's discussion)
9. Benchmark yields can move for microstructure reasons; using OTR yields as "curve points" can bias spreads. Use fitted curves/adjustments when appropriate
10. Convert microstructure advantages (price points) to yield equivalents using $DV01$: divide advantage by DV01 to get bp

### Cheat Sheet of Key Definitions + "How to Compute" Snippets

| Item | Formula / Definition |
|------|---------------------|
| OTR vs off-run | OTR = most recent issue; off-run = older |
| Repo repurchase price | $X_{rep} = X(1 + rd/360)$ |
| Special spread | $s = r_{GC} - r_{sp}$ |
| Financing advantage per 100 | $(P + AI) \cdot s \cdot d/360$ |
| Yield-equivalent advantage | $\text{bp} \approx \text{Adv}_{100}/DV01$ |
| Price from yield move | $\Delta P \approx -DV01 \cdot \Delta y_{\text{bp}}$ |
| Forward price from repo (no coupons) | $P_{fwd} + AI(d) = (P(0) + AI(0))(1 + rd/360)$ |
| Gross basis | $GB = P - cf \cdot F$ |
| Net basis | $NB = GB - \text{carry}$ |

---

### 25 Flashcards (Q/A)

**Q1:** What is "on-the-run" in Treasuries?
**A:** The most recently issued Treasury of a maturity sector.

**Q2:** Why are OTR issues often more liquid?
**A:** Benchmark and hedging demand concentrates trading in the latest issue; off-the-runs trade less.

**Q3:** Define repo in one line.
**A:** A sale of securities with an agreement to repurchase later at a higher price, implying an interest rate.

**Q4:** What is the basic repo interest formula used in the book?
**A:** Repurchase price $= X(1 + rd/360)$.

**Q5:** What is GC repo?
**A:** Repo where collateral can be any security in a broad acceptable class (e.g., any Treasury).

**Q6:** What is a "special" repo?
**A:** Repo against a particular security; rate often below GC when that security is in demand.

**Q7:** Define special spread $s$.
**A:** $s = r_{GC} - r_{sp}$.

**Q8:** Why might a special repo rate be low?
**A:** Market participants accept low interest to obtain a particular bond as collateral (high demand to borrow it).

**Q9:** What are the two components of the OTR liquidity premium in the book?
**A:** Financing advantage and pure liquidity premium.

**Q10:** What is "financing advantage"?
**A:** Benefit from financing at below-GC rates using a bond as collateral (specialness).

**Q11:** How do you convert a financing advantage into yield-equivalent bp?
**A:** Divide the price-point advantage by $DV01$.

**Q12:** What is DV01?
**A:** Price change per 1 bp change in yield (per $100 par for the bond-level DV01).

**Q13:** Why can OTR yields distort curve fitting?
**A:** They embed liquidity premiums and special financing effects beyond the smooth term structure.

**Q14:** What are common remedies for OTR distortion in curve fitting?
**A:** Discard OTRs, weight them differently, or adjust yields for liquidity/financing effects.

**Q15:** Why can swap spreads using OTR Treasuries be misleading?
**A:** Because OTR yields include liquidity premiums; use fitted curves or adjust yields instead.

**Q16:** What is the auction-cycle pattern of special spreads described in the book?
**A:** Small after auctions; tend to rise and peak before the next auction.

**Q17:** How do reopenings relate to the auction cycle?
**A:** Auctions can be new issues or reopenings; the book's figure marks both, and reopenings can affect the special spread pattern differently.

**Q18:** What does "reverse repo" mean in the book's trading-desk language?
**A:** From the desk's perspective, lending cash to borrow particular bonds (repo from the counterparty viewpoint).

**Q19:** How does a short Treasury position interact with repo?
**A:** The short must borrow the bond (often via reverse repo) and may need to roll if not covered.

**Q20:** What is "covering a short"?
**A:** Purchasing and delivering securities that were sold and borrowed.

**Q21:** Why can special repo rates have a floor in the book's discussion?
**A:** Because failing can dominate paying negative special rates; special cannot go below 0, bounding special spread by GC.

**Q22:** How do you compute carry in the book's P&L framework?
**A:** Carry = interest income (accrual + coupons) minus financing cost (repo on invoice).

**Q23:** Why must RV signals consider bid-ask?
**A:** Trading costs can be comparable to small RV edges; the book illustrates costs in 32nds on large notionals.

**Q24:** What is the forward price identity linking repo and carry (no coupons)?
**A:** $P_{fwd} = P(0) - \text{Carry}$.

**Q25:** What is net basis (preview)?
**A:** $NB = P_{fwd} - cf \cdot F = GB - \text{carry}$.

---

## Mini Problem Set

*14 questions, increasing difficulty. Solution sketches provided for questions 1-7 only.*

---

**1.** A 5-year Treasury has $DV01 = 0.045$ per 100. If its yield falls by 6 bp, estimate the price change per 100.

**Sketch:** $\Delta P \approx -DV01 \cdot \Delta y = -0.045 \cdot (-6) = +0.27$ points.

---

**2.** You finance $200mm at repo rate 4.8% for 10 days. Compute interest cost (assume simple, 360).

**Sketch:** Cost $= 200{,}000{,}000 \cdot 0.048 \cdot 10/360 = 200{,}000{,}000 \cdot 0.001\overline{3} \approx \$266{,}667$.

---

**3.** A bond is special at 2.0% while GC is 5.0%. For a $100mm long funded position over 7 days, compute financing advantage.

**Sketch:** Advantage $= 100{,}000{,}000 \cdot (0.05 - 0.02) \cdot 7/360 = 100{,}000{,}000 \cdot 0.03 \cdot 0.019444 \approx \$58{,}333$.

---

**4.** Using the answer to (3), compute yield-equivalent advantage if $DV01_{\$}$ is $85,000 per bp.

**Sketch:** bp $\approx 58{,}333/85{,}000 \approx 0.69$ bp.

---

**5.** A trader measures a 5-year swap spread using OTR yield 4.70% and swap rate 5.05%. Then uses fitted curve yield 4.74%. What is the difference in measured swap spread?

**Sketch:** OTR spread = 35 bp; fitted spread = 31 bp; difference = 4 bp.

---

**6.** A short position must borrow a security at special repo 0.25% while GC is 4.75% for 14 days on $50mm. Compute the incremental cost vs GC.

**Sketch:** Cost $= 50{,}000{,}000 \cdot (0.0475 - 0.0025) \cdot 14/360 = 50{,}000{,}000 \cdot 0.045 \cdot 0.038\overline{8} \approx \$87{,}500$.

---

**7.** Using (16.7) with no coupon in the window: $P(0) = 100.00$, $AI(0) = 0.50$, $P_{fwd} = 100.10$, $AI(d) = 0.80$, $d = 30$. Solve for implied repo $r$.

**Sketch:**
- Left $= 100.10 + 0.80 = 100.90$
- Denom $= 100.00 + 0.50 = 100.50$
- Ratio $\approx 1.0039801$

$$r = \frac{360}{30} \times (1.0039801 - 1) \approx 12 \times 0.0039801 \approx 0.04776 = 4.78\%$$

---

**8.** Explain qualitatively why OTR/off-run yield spreads might widen during market stress even if the fitted curve slope is unchanged.

---

**9.** Construct a simple RV signal combining an OTR/off-run yield spread with a special spread. Explain how you would avoid double-counting (financing vs pure liquidity).

---

**10.** Suppose a new auction is a reopening. Based on the book's qualitative description, how might you expect the special spread dynamics to differ vs a brand-new issue? (Qualitative.)

---

**11.** Design a hedged trade to isolate changes in specialness from general rate moves. What hedge ratio would you use and why?

---

**12.** Describe an empirical test (data inputs and regressors) to estimate the sensitivity of OTR-off-run yield spread to special spread.

---

**13.** In a curve-fitting context, describe how you would "adjust" on-the-run yields for microstructure before fitting, consistent with the book's discussion. What quantities would you need?

---

**14.** (Preview) Using the basis identity $NB = GB - \text{carry}$, explain how a trader might interpret a persistently positive net basis in the presence of repo constraints.

---

## Source Map

### (A) Verified Facts

| Fact | Source |
|------|--------|
| Treasury auction mechanics (competitive/noncompetitive bids) | Tuckman Ch 15-16 (secondary source) |
| On-the-run vs off-the-run definitions and liquidity differences | Tuckman Ch 15-16 |
| Repo as collateralized loan; repurchase price formula | Tuckman Ch 15-16 |
| GC vs special repo definitions | Tuckman Ch 15-16 |
| Special spread definition and floor at 0 (fails logic) | Tuckman Ch 15-16 |
| OTR liquidity premium components (financing + pure liquidity) | Tuckman Ch 15-16 |
| Auction-cycle pattern for special spreads | Tuckman Ch 15-16 (empirical figure description) |
| OTR yields distort curve fitting; remedies proposed | Tuckman Ch 15-16 |
| Financing advantage to yield-equivalent conversion | Tuckman Ch 15-16 (explicit example) |
| Forward price from repo (Eq 16.7, 16.8) | Tuckman Ch 16 |
| Basis identities (Eq 20.10-20.12) | Tuckman Ch 20 |

### (B) Reasoned Inference

| Inference | Derivation Logic |
|-----------|------------------|
| Microstructure decomposition (curve + financing + liquidity) | Logical combination of the two liquidity-premium components and carry/forward mechanics |
| Hedging with OTR exposes to liquidity/specialness shocks | Direct implication of OTR yield containing non-fundamental components |
| Time-varying liquidity premium | Implied by auction-cycle patterns in specialness and curve-fitting complications |

### (C) Speculation

| Item | Uncertainty |
|------|-------------|
| Exact U.S. Treasury auction calendar/timeline | Not enumerated in provided sources |
| Modern fails charges and penalty mechanics | Not detailed beyond the floor logic |
| CTD selection and delivery option details | Only basis definitions used; full machinery not developed here |
| Empirical sign/magnitude of when-issued cheapening | Would require specific empirical source |

---

*Last Updated: January 2026*
