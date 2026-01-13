# Chapter 9: Repo as the Bond Market's Funding Engine

---

## Fact Classification

### (A) Verified Facts (Source-Backed)

- Repos are used to finance dealer inventories and are economically equivalent to collateralized borrowing/lending; the repurchase price exceeds the sale price by repo interest (Tuckman Ch 15-16)
- GC vs special repo rates exist; specials can be materially lower than GC, reflecting scarcity/short demand (Tuckman Ch 15-16)
- Haircuts and repricing provisions exist to protect the cash lender against borrower default plus adverse collateral moves (Tuckman)
- In repo-related shorting mechanics, the security lender continues to receive coupons; the security borrower must pass coupon payments through (a "manufactured" coupon) (Tuckman)
- Coupon dates affect invoice price and thus the economics of the cash leg (Tuckman)
- Carry is defined as interest income on the bond minus financing cost of holding it (Tuckman)
- Repo links spot and forward prices via no-arbitrage replication (Tuckman)

### (B) Reasoned Inference (Derived from A)

- Repo is the funding leg inside many relative-value trades: it converts a spot bond position into a funded position (long financed at repo) or a short position (bond borrowed via reverse repo)
- "Specialness" acts like an embedded benefit of holding a scarce bond because it lowers the financing cost (or raises the investment return) relative to GC
- Repo-rate sensitivity of P&L follows directly from the funding cost formula
- Dollar valuation of specialness = principal × spread × daycount

### (C) Speculation (Clearly Labeled)

- Specific operational rules for UST fails charges, tri-party mechanics, and modern margining frequencies are market- and contract-dependent and are not fully specified in the provided sources
- "Open repo" contract definition and termination mechanics are not detailed in the provided sources
- Bilateral vs tri-party repo institutional workflows are not fully specified

---

## Conventions & Notation

### Default Market Context

Unless stated otherwise: U.S. Treasury-style repo used by dealers to finance/borrow securities. Repo is treated economically as secured borrowing/lending even though legal form may be a securities sale-and-repurchase.

### Repo Rate Quoting

Simple interest with day-count = ACT/360 for worked examples.

**Market-practice warning:** Repo day-count conventions vary by market and contract. In Tuckman's examples, repo interest is computed with a /360 convention, and repo and bond coupon accrual day counts can differ.

### Bond Pricing Notation

| Symbol | Definition |
|--------|------------|
| $P$ | Flat/clean price (excludes accrued) |
| $AI$ | Accrued interest |
| $I = P + AI$ | Invoice/full/dirty price (what actually changes hands on settlement) |

### Units

| Convention | Details |
|------------|---------|
| Prices $P$, $AI$, $I$ | Expressed per \$1 of face in formulas (so "103%" means 1.03), but examples often show "per \$100 face" |
| Rates | Decimal per annum (e.g., 5% = 0.05) |
| $d$ | Actual days in the repo term; year fraction is $d/360$ under ACT/360 |

### Notation Glossary

| Symbol | Definition |
|--------|------------|
| $N$ | Face value / par amount of the bond (dollars) |
| $P(t)$ | Flat (clean) price per \$1 face at time $t$ |
| $AI(t)$ | Accrued interest per \$1 face at time $t$ |
| $I(t) = P(t) + AI(t)$ | Invoice (dirty) price per \$1 face at time $t$ |
| $L_0$ | Cash lent/borrowed at repo start ("principal" of the cash leg) |
| $L_1$ | Cash repaid/received at repo end (repurchase cash) |
| $r$ | Repo rate (simple annual rate) |
| $d$ | Number of days from repo start to repo end |
| $h$ | Haircut (fraction of collateral value not financed), $0 \leq h < 1$ |
| GC | General collateral repo (collateral is "any of a broad acceptable set") |
| $r_{\text{GC}}$ | GC repo rate |
| $r_{\text{spec}}$ | Special repo rate for a particular security (often lower than GC) |
| $s = r_{\text{GC}} - r_{\text{spec}}$ | "Specialness" spread (annualized) |
| $C$ | Coupon payment amount on the bond (dollars) on a coupon date |

---

## Setup

### Conventions Used in This Chapter

- Repo is modeled as a two-leg transaction: initial exchange of cash vs collateral and a closing exchange at a repurchase price that embeds financing at the repo rate
- Default numeric convention for examples: ACT/360, simple interest, settlement on stated start/end dates
- Haircuts and repricing (margining): included conceptually and in examples, even though some repo examples in the sources ignore them for simplicity
- **Bilateral vs tri-party repo:** I'm not sure — the provided sources do not lay out a clean side-by-side institutional description

---

## Core Concepts

### 1) Repurchase Agreement (Repo) vs Reverse Repo

**Formal Definition:**

A **repo** is a transaction where Party A sells a security to Party B for cash today and simultaneously agrees to repurchase the same (or specified) security at a later date for a higher price. The difference is repo interest.

A **reverse repo** is the same transaction from the cash lender's perspective: Party B buys the security today and agrees to resell it later.

**Intuition:**

- Economically, repo is a secured loan: the borrower receives cash and posts the security as collateral; the lender provides cash and holds collateral as protection
- Repo can also be viewed as a forward contract on the security with repurchase price equal to the forward price (under the repo's embedded financing rate)

**Trading/Risk Practice:**

- Dealers "sell the repo" to finance long bond inventory (borrow cash against bonds rather than tying up balance sheet)
- Traders "buy the repo" (reverse repo) to obtain a bond as collateral — critically important for shorting specific securities

---

### 2) Start Leg vs End Leg (Cashflows and Collateral)

**Formal Definition:**

- **Start leg (initiation date):** cash vs security exchange
  - In a repo: borrower delivers collateral and receives cash $L_0$
  - In a reverse repo: lender delivers cash $L_0$ and receives collateral
- **End leg (maturity date):** collateral vs cash re-exchange at repurchase amount $L_1$

**Intuition:**

Think of $L_0$ as "principal" and $L_1 - L_0$ as "interest" on a secured money-market loan.

**Trading/Risk Practice:**

Settlement timing and fails matter: missing collateral delivery breaks the intended funding and can create P&L noise or forced closeouts.

---

### 3) Overnight, Term, and Open Repo

**Formal Definition:**

- **Overnight repo:** Start today, end next business day
- **Term repo:** Maturity beyond overnight (e.g., one week, one month)
- **Open repo:** I'm not sure — the provided sources discuss overnight and term repo explicitly, but do not define "open repo" mechanics (e.g., indefinite maturity with daily rate reset/cancel notice)

**Intuition:**

Overnight repo is the "daily funding heartbeat." Term repo locks in funding for a horizon, reducing rollover risk but potentially at a different rate.

**Trading/Risk Practice:**

**Rollover risk:** Funding long positions by rolling overnight exposes you to changes in tomorrow's repo rate and to the risk that funding disappears.

---

### 4) Collateral and Admissibility (GC vs Specials)

**Formal Definition:**

- **General collateral (GC):** Repo where the cash lender accepts any security from a broad eligible set (e.g., many Treasuries). The repo rate is a "general" secured funding rate
- **Special collateral (specials):** Repo where the cash lender demands a particular security. The repo rate for that security can be below GC

**Intuition:**

GC is about funding cash; specials are about sourcing a particular bond. If many shorts need the same bond, borrowers will "pay up" by accepting a lower (more favorable-to-cash-borrower) repo rate.

**Trading/Risk Practice:**

Specials drive relative value (e.g., on-the-run vs off-the-run): a bond can look "rich" in yield partly because it finances cheaply (special).

---

### 5) Specialness: The Economics of Scarcity and Short Demand

**Formal Definition:**

Define specialness spread:

$$\boxed{s = r_{\text{GC}} - r_{\text{spec}}}$$

Larger $s$ means more "special."

**Intuition:**

Owning a special bond is valuable because it embeds a financing advantage: you can borrow cash more cheaply against it (or, for shorts, you can source it more reliably).

**Trading/Risk Practice:**

Specialness can compress/expand quickly around auctions, index events, or crowded shorts, changing carry and forward pricing.

---

### 6) Haircuts, Repricing, and Margining

**Formal Definition:**

- A **haircut** is a reduction applied to collateral value for lending purposes (e.g., collateral worth \$100 with 10% haircut supports \$90 of cash lending)
- **Repricing (variation margin)** provisions require additional collateral when prices fall (and allow collateral return when prices rise)

**Intuition:**

Haircuts protect the cash lender against "default + collateral drop" before liquidation. Repricing turns a term repo into something closer to a daily-margined exposure to collateral value.

**Trading/Risk Practice:**

- Haircuts determine maximum leverage
- Repricing creates margin call liquidity risk

---

### 7) Repo and the Decomposition of Bond P&L: Price Change vs Carry

**Formal Definition:**

**Carry** is defined as interest income on the bond minus financing cost of holding it.

**Intuition:**

A bond can have positive carry even if its price doesn't move — because coupons accrue while funding costs accrue.

**Trading/Risk Practice:**

Carry is the "engine" behind many RV trades (carry/roll-down). Repo determines whether that engine is strong, weak, or negative.

---

### 8) Repo as a Forward-Pricing Link ("Implied Repo")

**Formal Definition:**

Under no-arbitrage replication arguments, a bond forward can be replicated by buying the bond spot and financing it at repo, implying a forward price relationship (with adjustments for accrued and coupons).

**Intuition:**

Repo converts spot ownership into a "locked-in" future cost of carrying the bond to delivery.

**Trading/Risk Practice:**

"Implied repo" is a diagnostic: compare forward prices (or futures basis) to financing + carry to spot mispricings or specialness.

---

## Math and Derivations

### 1) Repo Cashflows and Repurchase Price (Simple-Interest Repo)

**Assumptions:**
- Simple interest repo rate $r$ with ACT/360
- No interim coupon payments and no margining (for this basic derivation)

Let $L_0$ be the cash exchanged at the start leg. Then the repurchase cash $L_1$ satisfies:

$$\boxed{L_1 = L_0 \left(1 + r \frac{d}{360}\right)}$$

**Unit Check:**
- $r$ is per year, $d/360$ is years ⇒ $r(d/360)$ is dimensionless ⇒ $L_1$ is dollars ✓

**Sanity Checks:**
- If $d = 0$, then $L_1 = L_0$
- If $r$ increases, $L_1$ increases linearly for fixed $d$

---

### 2) Mapping to Bond Invoice Price

In many bond repos, $L_0$ is closely linked to the invoice price of the collateral:

$$L_0 \approx N \cdot I(0) = N \cdot (P(0) + AI(0))$$

subject to haircut and contract terms.

---

### 3) Haircut Mechanics (One Standard Convention)

**Assumption (explicit):** Haircut $h$ is applied as a percentage reduction of collateral market value to determine loan cash:

$$\boxed{L_0 = (1 - h) \, N \, I(0)}$$

**Alternative conventions warning:** Some markets define haircut as (Collateral Value − Loan)/Loan rather than (Collateral Value − Loan)/Collateral Value. If you want that definition, formulas must be restated.

**Implications:**

Equity (unfunded portion) posted by the repo borrower:

$$E_0 = N \, I(0) - L_0 = h \, N \, I(0)$$

Leverage (collateral value divided by equity):

$$\boxed{\text{Leverage} = \frac{N \, I(0)}{E_0} = \frac{1}{h}}$$

So a 2% haircut implies ~50× leverage (before additional constraints).

---

### 4) Carry and Funding-Adjusted Bond P&L (Tuckman's Decomposition)

Let $d$ be the holding horizon in days. The funding-adjusted P&L of buying the bond and selling after $d$ days, financed at repo, can be written:

$$\boxed{\text{P\&L} = N(P(d) + AI(d)) - N(P(0) + AI(0))\left(1 + r\frac{d}{360}\right)}$$

Rearranging gives:

$$\text{P\&L} = \underbrace{N(P(d) - P(0))}_{\text{Price change}} + \underbrace{N(AI(d) - AI(0))}_{\text{Interest income}} - \underbrace{N(P(0) + AI(0))\left(r\frac{d}{360}\right)}_{\text{Financing cost}}$$

**Interpretation:**
- **Price change** = $N(P(d) - P(0))$
- **Interest income** ≈ $N(AI(d) - AI(0))$ (coupon accrual)
- **Financing cost** = $N(P(0) + AI(0)) \, r(d/360)$
- **Carry** = Interest income − Financing cost

**Unit Check:** Each term is dollars ✓

**Sanity Check:** If prices don't move ($P(d) = P(0)$) and there's no accrual ($AI(d) = AI(0)$), P&L is negative and equals financing cost.

---

### 5) Repo Rate Risk ("Funding DV01" Idea)

Differentiate the P&L with respect to repo rate $r$ (holding other terms fixed):

$$\boxed{\frac{\partial \text{P\&L}}{\partial r} = -N(P(0) + AI(0)) \frac{d}{360}}$$

So a 1 bp increase in repo rate reduces P&L by approximately:

$$\Delta \text{P\&L} \approx -N(P(0) + AI(0)) \frac{d}{360} \times 10^{-4}$$

---

### 6) Forward Link: Repo-Implied Forward Invoice Price (No Intermediate Coupon)

A standard no-arbitrage replication is:
- Long forward ≈ buy bond spot and sell the repo (finance) to delivery

This yields the forward invoice price relationship:

$$\boxed{P_{\text{fwd}} + AI(d) = (P(0) + AI(0))\left(1 + r\frac{d}{360}\right)}$$

and thus:

$$P_{\text{fwd}} = (P(0) + AI(0))\left(1 + r\frac{d}{360}\right) - AI(d)$$

**Sanity Check:** If the bond is a zero-coupon bond ($AI = 0$), this reduces to $P_{\text{fwd}} = P(0)(1 + rd/360)$.

---

### 7) Forward Link with an Intermediate Coupon (Two Subperiods)

If there is a coupon payment during the carry period:
- The bond owner (security lender) continues to receive coupons
- The security borrower must pass through coupon payments, and it is economically consistent to reduce the repo loan balance by the coupon amount on the coupon date

Let:
- $d_1$ = days from start to coupon date
- $d_2$ = days from coupon date to delivery/end
- $C$ = coupon paid at coupon date

Then the forward invoice price satisfies:

$$\boxed{P_{\text{fwd}} + AI(d) = \left((P(0) + AI(0))\left(1 + r\frac{d_1}{360}\right) - C\right)\left(1 + r\frac{d_2}{360}\right)}$$

**Sanity Checks:**
- If $C = 0$, this collapses to the no-coupon formula
- Coupon reduces the forward invoice price relative to a no-coupon world (all else equal)

---

## Measurement & Risk (Chapter 9 Scope)

### How Repo Impacts Carry/Returns and "Funding-Adjusted" P&L

- **Carry depends on repo:** A higher repo rate increases financing cost and reduces carry (see the P&L decomposition)
- **Funding is applied to the invoice price, not the face value.** This matters because coupon accrual is on face, but financing cost is on full price
- **Funding-adjusted returns** on levered bond positions can be dominated by small changes in repo rates when haircuts are low (high leverage)

### Repo Rate Risk vs Collateral Price Risk

| Risk Type | Description |
|-----------|-------------|
| **Repo rate risk** | Uncertainty in $r$ (especially when rolling overnight) impacts the financing cost leg |
| **Collateral price risk** | The bond's market value changes; with margining, price declines can trigger margin calls |

In a funded long bond, price risk typically dwarfs repo-rate risk over short horizons, but liquidity constraints make repo-rate and margining risk decisive in stress.

### Haircut/Margin and Liquidity Risk

- Haircuts limit leverage but also create a "liquidity buffer requirement" (equity tied up)
- Repricing/variation margin converts market moves into cash demands; inability to meet them can force liquidation

### Specialness and Shorting Constraints as Risk Drivers

Specials create a funding benefit to holding a specific bond but also expose shorts to:
- Risk of that bond becoming "more special" (repo rate drops, specialness spread widens)
- Inability to source the bond (short squeeze mechanics)

Special financing can meaningfully distort relative-value comparisons unless explicitly adjusted (e.g., comparing on a forward basis).

### Practical "What Breaks" Scenarios

| Scenario | Description |
|----------|-------------|
| **Fails** | Inability to deliver the security on settlement; can dominate the economics of very special collateral |
| **Margin calls** | Collateral value drops ⇒ required additional collateral or cash; if not met, position may be closed out |
| **Collateral substitution** | I'm not sure — contractual right to substitute collateral is not developed in the provided sources |

---

## Worked Examples

### Example A — Basic Repo Cashflows: Repurchase Price and Implied Interest

**Goal:** Given bond price, repo rate, start/end dates, day count, compute repurchase cash and implied interest.

**Conventions:**
- Day count: ACT/360
- Repo interest: simple
- Settlement: start and end as stated
- Coupon treatment: none (repo does not span a coupon date)
- Haircut: none

**Inputs:**
- Face $N = \$10{,}000{,}000$
- Clean price $P = 102.50$ per \$100 face
- Accrued $AI = 1.20$ per \$100 face
  ⇒ Invoice price $I = 103.70$ per \$100 face
- Repo start: Jan 2; repo end: Jan 16 ⇒ $d = 14$ days
- Repo rate $r = 4.20\% = 0.042$

**Step 1: Compute initial cash (start leg)**

Invoice value:
$$L_0 = N \times \frac{I}{100} = 10{,}000{,}000 \times 1.037 = \$10{,}370{,}000$$

**Step 2: Compute repo interest**

$$\text{Interest} = L_0 \times r \frac{d}{360} = 10{,}370{,}000 \times 0.042 \times \frac{14}{360}$$

Compute:
- $0.042 \times 14 = 0.588$
- $0.588 / 360 = 0.001633\overline{3}$
- $10{,}370{,}000 \times 0.001633\overline{3} = \$16{,}937.67$

**Step 3: Repurchase cash (end leg)**

$$L_1 = L_0 + \text{Interest} = 10{,}370{,}000 + 16{,}937.67 = \$10{,}386{,}937.67$$

**Outputs:**
- Repurchase cash $L_1 \approx \$10{,}386{,}937.67$
- Implied repo interest over 14 days $\approx \$16{,}937.67$

---

### Example B — Overnight vs Term: Rolling O/N Repo vs a 5-Day Term Repo

**Goal:** Compare financing cost of rolling overnight repo vs locking a term repo.

**Conventions:**
- Day count: ACT/360
- Repo interest: simple per day
- Principal is assumed constant (close and reopen each day at same $L_0$)
- No coupon within 5 days

**Inputs:**
- Principal financed $L_0 = \$10{,}000{,}000$
- Horizon: 5 business days

**Case 1: Roll overnight repo**

Daily overnight repo rates:
| Day | Rate |
|-----|------|
| 1 | 4.00% |
| 2 | 4.10% |
| 3 | 4.05% |
| 4 | 4.20% |
| 5 | 4.15% |

Total interest:
$$\text{Interest}_{\text{ON}} = L_0 \sum_{i=1}^{5} \frac{r_i}{360}$$

Sum of rates:
$$0.040 + 0.041 + 0.0405 + 0.042 + 0.0415 = 0.205$$

So:
$$\text{Interest}_{\text{ON}} = 10{,}000{,}000 \times \frac{0.205}{360} = 10{,}000{,}000 \times 0.000569\overline{4} = \$5{,}694.44$$

**Case 2: 5-day term repo**

Term repo rate $r_{\text{term}} = 4.12\% = 0.0412$, $d = 5$:

$$\text{Interest}_{\text{term}} = L_0 \times r_{\text{term}} \frac{5}{360} = 10{,}000{,}000 \times 0.0412 \times \frac{5}{360}$$

Compute:
- $0.0412 \times 5 = 0.206$
- $0.206 / 360 = 0.000572\overline{2}$
- Interest $= 10{,}000{,}000 \times 0.000572\overline{2} = \$5{,}722.22$

**Comparison:**
| Strategy | Cost |
|----------|------|
| Rolling O/N | $\approx \$5{,}694.44$ |
| Term repo | $\approx \$5{,}722.22$ |
| **Difference** | $\approx \$27.78$ |

**Interpretation:**
- Rolling overnight exposes you to day-to-day rate changes (funding risk), even if the expected total cost is similar
- Term repo reduces rollover risk but may embed a premium/discount depending on funding conditions

---

### Example C — Haircut Mechanics: Cash Raised, Leverage, ROE, and a Margin Call

**Goal:** Show how a haircut changes cash raised, leverage, and ROE; include a mark-to-market + margin call example.

**Conventions:**
- Haircut definition: cash lent equals $(1 - h)$ times collateral market value
- Day count: ACT/360 (repo interest not the focus here)
- Margining: daily repricing to maintain the haircut
- Coupon: ignored for simplicity

**Inputs:**
- Face $N = \$100{,}000{,}000$
- Invoice price at start $I_0 = 103.00\%$ ⇒ collateral market value:
$$MV_0 = 100{,}000{,}000 \times 1.03 = \$103{,}000{,}000$$
- Haircut $h = 2\% = 0.02$

**Step 1: Cash raised and equity**

$$L_0 = (1 - h) \, MV_0 = 0.98 \times 103{,}000{,}000 = \$100{,}940{,}000$$

Equity tied up:
$$E_0 = MV_0 - L_0 = 103{,}000{,}000 - 100{,}940{,}000 = \$2{,}060{,}000$$

**Step 2: Leverage**

$$\text{Leverage} = \frac{MV_0}{E_0} = \frac{103{,}000{,}000}{2{,}060{,}000} = 50.0\times$$

(As predicted by $1/h$.)

**Step 3: Mark-to-market shock and margin call**

Suppose the bond invoice price falls from 103% to 101% (a 2-point drop):

$$MV_1 = 100{,}000{,}000 \times 1.01 = \$101{,}000{,}000$$

To maintain a 2% haircut, the maximum permitted loan is:

$$L_{1,\max} = (1 - h) \, MV_1 = 0.98 \times 101{,}000{,}000 = \$98{,}980{,}000$$

Outstanding loan is still $L_0 = 100{,}940{,}000$, so the margin call is:

$$\boxed{\text{Margin call} = L_0 - L_{1,\max} = 100{,}940{,}000 - 98{,}980{,}000 = \$1{,}960{,}000}$$

**Step 4: Equity impact**

Equity before meeting margin:
$$E_{1,\text{pre}} = MV_1 - L_0 = 101{,}000{,}000 - 100{,}940{,}000 = \$60{,}000$$

So a 2-point adverse move almost wipes out the original \$2.06m equity buffer.

After meeting the margin call (repaying \$1.96m), loan becomes \$98.98m and equity becomes:
$$E_{1,\text{post}} = MV_1 - 98{,}980{,}000 = \$2{,}020{,}000$$

which is exactly 2% of $MV_1$.

**Step 5: ROE intuition (simple)**

If net carry on the collateral were +1.00% per year on market value:
$$\text{Carry} \approx 1.00\% \times 103{,}000{,}000 = \$1{,}030{,}000 \text{ per year}$$

ROE on $E_0 = \$2.06\text{m}$:
$$\text{ROE} \approx \frac{1.03\text{m}}{2.06\text{m}} \approx 50\% \text{ per year}$$

This matches the leverage intuition: small net spreads become large ROE when haircuts are small — until margin calls force deleveraging.

---

### Example D — Coupon During Repo: Repo Spanning a Coupon Date with Cashflow Reconciliation

**Goal:** Illustrate a repo spanning a coupon date — show how coupon/price/accrued are handled under a stated convention, and reconcile cashflows.

**Conventions:**
- Day count: ACT/360
- Repo interest: simple
- Coupon handling convention:
  - The bond owner / security lender continues to receive coupons
  - The security borrower must make a manufactured coupon payment on coupon date
  - Since a coupon payment reduces the bond's invoice price by exactly the coupon, it is economically consistent to reduce the repo loan balance by the coupon amount on the coupon date
- Haircut: none (for clarity)

**Inputs:**
- Face $N = \$10{,}000{,}000$
- Invoice value at repo start: $L_0 = \$10{,}500{,}000$ (assume $I_0 = 105\%$)
- Repo rate $r = 3.60\% = 0.036$
- Repo start: Jan 10
- Coupon date: Feb 15
- Repo end: Feb 20
- Days: $d_1 = 36$ (Jan 10 → Feb 15), $d_2 = 5$ (Feb 15 → Feb 20)
- Coupon rate: 6% annual, semiannual coupon = 3% of face
  $$C = 0.03 \times 10{,}000{,}000 = \$300{,}000$$

**Step 1: Accrue repo interest up to coupon date**

$$L_{\text{pre-cpn}} = L_0 \left(1 + r\frac{d_1}{360}\right) = 10{,}500{,}000 \left(1 + 0.036 \times \frac{36}{360}\right)$$

Compute:
- $0.036 \times 36/360 = 0.036 \times 0.1 = 0.0036$
- $L_{\text{pre-cpn}} = 10{,}500{,}000 \times 1.0036 = \$10{,}537{,}800$

**Step 2: Coupon paid through, loan balance reduced**

On Feb 15, borrower pays coupon $C = 300{,}000$ to the security lender, and (by convention) loan balance is reduced by $C$:

$$L_{\text{post-cpn}} = L_{\text{pre-cpn}} - C = 10{,}537{,}800 - 300{,}000 = \$10{,}237{,}800$$

**Step 3: Accrue repo interest from coupon date to repo end**

$$L_1 = L_{\text{post-cpn}} \left(1 + r\frac{d_2}{360}\right) = 10{,}237{,}800 \left(1 + 0.036 \times \frac{5}{360}\right)$$

Compute:
- $0.036 \times 5/360 = 0.036 \times 0.01388\overline{8} = 0.0005$
- Interest over second period $= 10{,}237{,}800 \times 0.0005 = \$5{,}118.90$
- Repurchase cash: $L_1 = 10{,}237{,}800 + 5{,}118.90 = \$10{,}242{,}918.90$

**Cashflow Reconciliation (from the security borrower's perspective):**

| Date | Cashflow |
|------|----------|
| Jan 10 | +\$10,500,000 (start leg, receive cash) |
| Feb 15 | −\$300,000 (manufactured coupon passed through) |
| Feb 20 | −\$10,242,918.90 (end leg, repay repo cash) |

Net cash to borrower across the life:
$$10{,}500{,}000 - 300{,}000 - 10{,}242{,}918.90 = -\$42{,}918.90$$

**Interpretation:** Total cost equals repo financing interest (over both subperiods) net of the coupon's principal reduction effect.

---

### Example E — GC vs Special: Financing Benefit in Dollars

**Goal:** Use two repo rates (GC and special) to compute the financing benefit/cost of borrowing a specific security; interpret "specialness" in \$ terms.

**Conventions:**
- Day count: ACT/360
- Simple interest
- Compare funding cost of financing the same invoice principal at different repo rates

**Inputs:**
- Principal financed $L_0 = \$100{,}000{,}000$
- GC repo rate $r_{\text{GC}} = 5.44\% = 0.0544$
- Special repo rate $r_{\text{spec}} = 3.85\% = 0.0385$
- Horizon $d = 1$ day

**Step 1: Compute 1-day interest cost at GC**

$$\text{Cost}_{\text{GC}} = L_0 \, r_{\text{GC}} \frac{1}{360} = 100{,}000{,}000 \times \frac{0.0544}{360}$$

Compute:
- $0.0544 / 360 = 0.000151\overline{1}$
- Cost $\approx 100{,}000{,}000 \times 0.000151\overline{1} = \$15{,}111.11$

**Step 2: Compute 1-day interest cost at special**

$$\text{Cost}_{\text{spec}} = 100{,}000{,}000 \times \frac{0.0385}{360}$$

Compute:
- $0.0385 / 360 = 0.000106\overline{9}$
- Cost $\approx \$10{,}694.44$

**Step 3: Financing benefit of specialness**

$$\boxed{\text{Benefit} = \text{Cost}_{\text{GC}} - \text{Cost}_{\text{spec}} \approx 15{,}111.11 - 10{,}694.44 = \$4{,}416.67 \text{ per day}}$$

Specialness spread:
$$s = r_{\text{GC}} - r_{\text{spec}} = 0.0544 - 0.0385 = 0.0159 = 159 \text{ bp}$$

Dollar benefit check:
$$100{,}000{,}000 \times \frac{0.0159}{360} = \$4{,}416.67 \checkmark$$

**Interpretation:**
"Special" collateral is effectively worth ~\$4.4k per \$100mm per day in financing advantage versus GC under these rates.

---

### Example F — Fails vs Paying Special: A Toy Break-Even Comparison

**Goal:** Compare the economics of failing vs borrowing "very special" collateral; show break-even logic.

**Conventions and Limitations:**
- The provided sources motivate a key bound: if the special repo rate were driven below 0%, a short would rationally prefer to fail rather than borrow at a negative rate
- Hence special repo rates are bounded below by ~0% and specialness spread is bounded above by ~GC
- I'm not sure about any specific modern "fails charge" formula from the provided sources, so we keep the comparison stylized and rate-based

**Toy Setup:**
- A trader is short a bond and must deliver it today to receive sale proceeds $S = \$100{,}000{,}000$
- If the trader fails for 1 day, assume (stylized) they effectively earn 0% on the sale proceeds
- If the trader can borrow the bond via a special repo at rate $r_{\text{spec}}$, they can deliver, receive $S$, and the cash they lend in the reverse repo earns $r_{\text{spec}}$

**Inputs:**
- Sale proceeds $S = \$100{,}000{,}000$
- GC repo rate $r_{\text{GC}} = 2.00\% = 0.0200$
- Consider two special rates:
  - Case (i): $r_{\text{spec}} = 0.10\% = 0.0010$ (very special)
  - Case (ii): $r_{\text{spec}} = 0.00\% = 0.0000$ (extremely special)

**Case (i): Borrow at 0.10% vs fail**

Interest earned by delivering (and lending cash at $r_{\text{spec}}$) over 1 day:
$$\text{Earn}_{\text{deliver}} = S \, r_{\text{spec}} \frac{1}{360} = 100{,}000{,}000 \times \frac{0.0010}{360} = \$277.78$$

Fail earns (stylized) \$0.

So delivering dominates failing by **\$277.78/day per \$100mm**.

**Case (ii): Borrow at 0% vs fail**

$$\text{Earn}_{\text{deliver}} = 100{,}000{,}000 \times \frac{0}{360} = \$0$$

Fail also earns \$0 ⇒ **indifferent**.

**Break-Even Logic:**
Under this stylized model, failing is weakly preferred whenever $r_{\text{spec}} < 0\%$ because delivering would force you to "lend cash" at a negative rate (i.e., pay to lend), which is worse than earning zero by failing.

Therefore $r_{\text{spec}}$ is bounded below by ~0% and specialness $s = r_{\text{GC}} - r_{\text{spec}}$ is bounded above by ~$r_{\text{GC}}$ in this framing.

---

### Example G — Implied Repo / Forward Link: Relate a Bond Forward Price to Repo Financing and Carry

**Goal:** Show how a bond forward price can be related to repo financing and carry via a simplified no-arbitrage relationship.

**Conventions:**
- Day count: ACT/360
- Repo interest: simple
- There is one coupon payment during the forward horizon
- Coupon handling: coupon is paid to the bond owner; the cash/loan balance is reduced by the coupon on coupon date
- We compute a forward clean price from a forward invoice relationship

**Inputs (per \$100 face unless noted):**
- Spot clean price $P(0) = 109.00$
- Spot accrued $AI(0) = 0.50$
  ⇒ Spot invoice $P(0) + AI(0) = 109.50$
- Repo rate $r = 2.40\% = 0.024$
- Days to coupon: $d_1 = 30$
- Days from coupon to delivery: $d_2 = 60$
- Semiannual coupon payment: $C = 2.00$ per \$100 face
- Accrued at delivery: delivery occurs 60 days into the new coupon period of assumed length $D = 182$ days

$$AI(d) = C \times \frac{60}{182} = 2.00 \times \frac{60}{182}$$

Compute:
- $60/182 = 0.329670...$
- $AI(d) = 2.00 \times 0.329670... = 0.65934$

**Step 1: Grow invoice to coupon date at repo**

$$\text{Invoice at coupon date (before coupon)} = 109.50 \left(1 + 0.024 \times \frac{30}{360}\right)$$

Compute:
- $0.024 \times 30/360 = 0.024 \times 1/12 = 0.002$
- Multiply: $109.50 \times 1.002 = 109.719$

**Step 2: Subtract coupon (passed through / reduces loan balance)**

$$109.719 - 2.00 = 107.719$$

**Step 3: Grow to delivery**

$$\text{Forward invoice} = 107.719 \left(1 + 0.024 \times \frac{60}{360}\right)$$

Compute:
- $0.024 \times 60/360 = 0.024 \times 1/6 = 0.004$
- Multiply: $107.719 \times 1.004 = 108.149876$

So:
$$P_{\text{fwd}} + AI(d) = 108.149876$$

**Step 4: Solve for forward clean price**

$$\boxed{P_{\text{fwd}} = 108.149876 - 0.65934 = 107.490536}$$

**Output:**
- Forward clean price ≈ **107.4905** per \$100 face
- Forward invoice price ≈ **108.1499** per \$100 face

**Interpretation:**
The forward is below spot clean (109.00 → 107.49) because (in this setup) coupon income plus low repo financing produces positive carry, which lowers the forward clean price once accrued-at-delivery is accounted for.

---

## Practical Notes

### Market Structure: Bilateral vs Tri-Party

I'm not sure — the provided sources focus on repo economics (financing, GC/specials, haircuts conceptually) rather than detailing bilateral vs tri-party workflows. If you want this, we should specify the market (UST vs other) and add a short appendix from an approved repo-market reference.

### Common Quoting/Contract Gotchas

| Issue | Notes |
|-------|-------|
| **Rate quoting (simple vs compounded)** | The repo examples use simple interest conventions. If your market uses compounding, adjust $L_1$ accordingly |
| **Day count (ACT/360 is common)** | Repo interest often uses /360; bond coupon accrual uses the bond's day count, which can differ |
| **Treatment of accrued interest and coupon payments** | Be explicit whether loan principal is based on clean or invoice price (sources emphasize invoice). If repo spans a coupon, clarify "manufactured coupon" and whether cash leg is reduced |
| **Settlement lags and calendar adjustments** | I'm not sure for specific business-day rules — getting dates wrong is a common source of funding P&L noise |

### Implementation Pitfalls

| Pitfall | Mitigation |
|---------|------------|
| **Date generation, holiday calendars, business-day rules** | Use a consistent calendar and day-count engine for repo and for bond accrual |
| **Sign conventions (repo vs reverse repo)** | Repo (sell repo): receive cash today, pay later (financing cost). Reverse repo (buy repo): pay cash today, receive later (interest income) |
| **Margining frequency and price source** | Haircuts and repricing create path-dependent liquidity needs; the mark source and timing (intraday vs EOD) matters in practice |

### Verification Tests

| Test | Description |
|------|-------------|
| **Cashflow reconciliation** | Initial cash + interest + coupon adjustments = repurchase cashflows |
| **Bounds/sanity** | Special repo rates should not be meaningfully negative (fail vs deliver bound). With haircut $h$, implied leverage near $1/h$ |
| **Sensitivity sanity** | Higher repo rate → higher financing cost; check $\partial \text{P\&L}/\partial r < 0$ for funded longs |

---

## Summary & Recall

### 10-Bullet Executive Summary

1. **Repo is the bond market's core secured funding mechanism:** cash vs collateral today, reversed later at a repurchase price embedding repo interest
2. **Repo vs reverse repo** are the same contract from opposite perspectives; sign conventions matter for P&L
3. **Dealers use repo** to finance inventories and avoid tying up scarce balance sheet capital
4. **GC repo** is secured funding against a broad collateral set; **specials** are repos against a particular bond, often at lower rates
5. **Specialness** ($r_{\text{GC}} - r_{\text{spec}}$) measures scarcity/short demand and has real dollar value via financing savings
6. **Haircuts** limit leverage but create equity "skin in the game" and are essential for lender protection
7. **Repricing/variation margin** converts collateral price moves into immediate liquidity needs (margin calls)
8. **Funding-adjusted P&L** decomposes into price change + carry, where carry = coupon accrual − financing cost
9. **Repo links spot and forward prices:** buying spot and financing at repo replicates a forward (with accrued/coupon adjustments)
10. **In very special collateral,** delivery fails can constrain how low special rates can go; operational frictions matter in stress

### Cheat Sheet of Formulas

| Formula | Meaning |
|---------|---------|
| $L_1 = L_0(1 + r \cdot d/360)$ | Repo repurchase cash |
| $I(t) = P(t) + AI(t)$ | Invoice price |
| $L_0 = (1-h) \, N \, I(0)$ | Haircut convention (chosen) |
| $E_0 = h \, N \, I(0)$ | Equity posted |
| Leverage $\approx 1/h$ | Implied leverage from haircut |
| $\text{P\&L} = N(P(d) + AI(d)) - N(P(0) + AI(0))(1 + rd/360)$ | Funding-adjusted P&L (funded long) |
| $\frac{\partial \text{P\&L}}{\partial r} = -N(P(0) + AI(0)) \frac{d}{360}$ | Repo-rate sensitivity |
| $P_{\text{fwd}} + AI(d) = (P(0) + AI(0))(1 + rd/360)$ | Forward invoice (no coupon) |
| $P_{\text{fwd}} + AI(d) = ((P(0) + AI(0))(1 + rd_1/360) - C)(1 + rd_2/360)$ | Forward invoice (one coupon) |
| $s = r_{\text{GC}} - r_{\text{spec}}$ | Specialness spread |
| Dollar benefit $\approx L_0 \, s \, d/360$ | Specialness in dollars |

---

## Flashcards (25 Q/A Pairs)

**Q1:** What is a repo economically?
**A1:** A secured loan: cash borrower posts securities as collateral and repays cash plus interest.

**Q2:** What is a reverse repo?
**A2:** The cash lender's side of a repo: lend cash, receive collateral, earn repo interest.

**Q3:** What are the two legs of a repo?
**A3:** Start leg (cash vs collateral) and end leg (repurchase: collateral returned vs cash repaid).

**Q4:** Why is the repurchase price higher?
**A4:** It embeds repo interest (the repo rate).

**Q5:** Define invoice (dirty) price.
**A5:** Clean price plus accrued interest.

**Q6:** Why does repo typically reference invoice price?
**A6:** Because settlement cash exchanged reflects full economic value including accrued.

**Q7:** Define GC repo.
**A7:** Repo against a broad eligible collateral set, reflecting general secured funding conditions.

**Q8:** Define special repo.
**A8:** Repo against a particular security, often at a lower rate due to scarcity/short demand.

**Q9:** What is "specialness"?
**A9:** The spread $r_{\text{GC}} - r_{\text{spec}}$.

**Q10:** How does specialness show up in dollars?
**A10:** Financing savings $\approx L_0 (r_{\text{GC}} - r_{\text{spec}}) \, d/360$.

**Q11:** Why do specials arise?
**A11:** Demand to borrow a specific bond (e.g., to short it) exceeds available supply.

**Q12:** What is a haircut?
**A12:** A reduction in collateral value used for lending, creating overcollateralization.

**Q13:** Why do haircuts exist?
**A13:** To protect the cash lender against borrower default and collateral price declines.

**Q14:** What is repricing/variation margin in repo?
**A14:** Adjusting collateral/cash to maintain agreed collateralization as prices change.

**Q15:** Define carry (funded bond).
**A15:** Coupon accrual (interest income) minus repo financing cost.

**Q16:** Write the basic repurchase cash formula.
**A16:** $L_1 = L_0(1 + rd/360)$ under simple ACT/360.

**Q17:** What is repo-rate risk in a funded long?
**A17:** P&L sensitivity to changes in financing rates, especially when rolling overnight.

**Q18:** What is collateral price risk in repo?
**A18:** Bond price moves can trigger margin calls and change equity/leverage.

**Q19:** How does a low haircut affect leverage?
**A19:** Leverage is approximately $1/h$; lower $h$ ⇒ higher leverage.

**Q20:** What happens if collateral price falls under repricing?
**A20:** The borrower must post more collateral or repay cash (margin call).

**Q21:** Who receives coupons during a repo?
**A21:** Economically, the bond owner/security lender continues to receive coupons.

**Q22:** What is a manufactured coupon?
**A22:** A coupon payment passed from the security borrower to the security lender during a borrow.

**Q23:** How does repo link to forward pricing?
**A23:** Buying spot and financing at repo replicates a forward, implying a no-arbitrage forward price.

**Q24:** What is implied repo?
**A24:** The financing rate implied by observed spot/forward (or spot/futures) pricing and carry.

**Q25:** Why can very negative special rates be unstable?
**A25:** Because failing to deliver can dominate borrowing at sufficiently negative rates (rate-based bound).

---

## Mini Problem Set (14 Questions)

### Problems (Solution Sketches for Q1–Q7)

**Problem 1:** (Repo cashflow) A repo starts with $L_0 = \$25{,}000{,}000$, $r = 3.75\%$, $d = 3$ days ACT/360. Compute $L_1$ and interest.

*Sketch:* Use $L_1 = L_0(1 + rd/360) = 25{,}000{,}000 \times (1 + 0.0375 \times 3/360)$. Interest $= L_0 \times 0.0375 \times 3/360$.

**Problem 2:** (Invoice vs clean) A bond has clean price 99.75 and accrued 1.10 (per \$100). For $N = \$50\text{mm}$, compute invoice cash exchanged on settlement.

*Sketch:* Invoice per \$100 is $99.75 + 1.10 = 100.85$ ⇒ cash $= 50{,}000{,}000 \times 1.0085 = \$50{,}425{,}000$.

**Problem 3:** (Carry sign) In a funded long, when is carry likely positive?

*Sketch:* When coupon accrual exceeds financing cost; often when coupon rate is sufficiently above repo rate, adjusting for invoice vs face and day counts.

**Problem 4:** (Repo-rate sensitivity) For $N = \$100\text{mm}$, invoice 102%, $d = 7$, compute approximate P&L change for a +10 bp move in repo rate.

*Sketch:* $\Delta \text{P\&L} \approx -N \times I(0) \times (d/360) \times \Delta r = -100{,}000{,}000 \times 1.02 \times (7/360) \times 0.0010$.

**Problem 5:** (Haircut & leverage) Collateral market value is \$80mm; haircut is 5%. Compute cash lent and leverage.

*Sketch:* $L_0 = 0.95 \times 80{,}000{,}000 = \$76{,}000{,}000$; equity $= 0.05 \times 80{,}000{,}000 = \$4{,}000{,}000$; leverage $\approx 1/0.05 = 20\times$.

**Problem 6:** (Margin call) Using Q5, if collateral falls to \$76mm and haircut stays 5%, compute margin call if loan is not yet adjusted.

*Sketch:* Required max loan $= 0.95 \times 76{,}000{,}000 = \$72{,}200{,}000$; margin call $= 76{,}000{,}000 - 72{,}200{,}000 = \$3{,}800{,}000$.

**Problem 7:** (Specialness in dollars) $r_{\text{GC}} = 4.5\%$, $r_{\text{spec}} = 0.5\%$, $L_0 = \$200\text{mm}$, $d = 1$. Compute daily dollar value of specialness.

*Sketch:* $L_0 (r_{\text{GC}} - r_{\text{spec}})/360 = 200{,}000{,}000 \times 0.04/360 = \$22{,}222.22$.

**Problem 8:** (Coupon in repo) Build a 40-day repo spanning a coupon. Show cashflows under a stated coupon/loan adjustment convention.

**Problem 9:** (Forward price no coupon) Given spot invoice and repo rate, compute forward clean price at $d$ days (given accrued at delivery).

**Problem 10:** (Forward price with coupon) Repeat Q9 but with one coupon inside the forward horizon.

**Problem 11:** (Rolling funding risk) Describe two distinct risks when funding a term trade by rolling overnight repo.

**Problem 12:** (Specialness risk) A bond becomes "more special" after you short it. How does that affect your P&L and funding?

**Problem 13:** (Stress scenario) Describe a plausible sequence: collateral selloff → margin calls → forced unwinds → further spread dislocations. Identify where repo is central.

**Problem 14:** (Design) Propose a set of internal checks for a repo P&L engine (inputs, sign conventions, and cashflow reconciliation) for both repo and reverse repo.

---

## Source Map

### Directly Source-Backed (Primary: Tuckman Ch 15-16)

| Content | Source |
|---------|--------|
| Repo as mechanism for financing dealer inventories; start/end legs and repurchase price logic | Tuckman Ch 15-16 |
| GC vs special repo; interpretation of specials and bounds related to delivery fails | Tuckman Ch 15-16 |
| Carry definition and funded bond P&L decomposition into price change and carry | Tuckman Ch 15-16 |
| Link between repo and forward pricing, including formulas for forward invoice price with/without intermediate coupon | Tuckman Ch 15-16 |
| Coupon treatment in repo/shorting: security lender receives coupons; borrower passes through | Tuckman Ch 15-16 |

### Directly Source-Backed (Secondary)

| Content | Source |
|---------|--------|
| Repo as key vehicle for financing large inventories; equivalence to forward/secured borrowing | Conceptual framing from sources |
| Haircut as reduction in collateral value for lending purposes | General collateral/margin concept |

### Derived (From Source-Backed Definitions and Algebra)

| Derivation | Logic |
|------------|-------|
| Repo-rate sensitivity of funded P&L ($\partial \text{P\&L}/\partial r$) | Direct differentiation of P&L formula |
| Dollar valuation of specialness as principal × spread × daycount | Arithmetic from spread definition |
| Worked numeric examples | Implementation of source formulas under explicit conventions |

### Uncertain / "I'm Not Sure"

| Topic | Notes |
|-------|-------|
| Detailed bilateral vs tri-party repo market plumbing | Not fully specified in provided sources |
| "Open repo" contract definition and termination mechanics | Not defined in provided sources |
| Modern fails-charge schedules and exact penalty formulas | Not specified in provided sources |
| Collateral substitution rights and standard legal documentation terms | Not developed in provided sources |

---

*Cross-references:*
- Bond pricing and invoice/clean/dirty: see Chapter 5
- Day count conventions: see Chapter 4
- Carry and rolldown: see Chapter 7
- Treasury futures and implied repo: see Chapter 23
