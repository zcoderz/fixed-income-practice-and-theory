# Chapter 4: Money-Market Building Blocks (The Shortest Curve Points)

---

## Introduction

Every yield curve begins somewhere. Before you can bootstrap to 30-year swap rates or price a 10-year Treasury bond, you need the first few discount factors—the anchors that pin down the present value of the nearest cashflows. These anchors come from the **money market**: deposits, Treasury bills, forward rate agreements (FRAs), and short-term interest rate futures.

The money-market segment of the curve looks deceptively simple. Instruments mature in weeks or months, not years. The math involves little more than present value with simple interest. Yet this is precisely where **day-count conventions matter most**: a one-day difference in accrual fraction can be as significant as a basis point difference in rate. At the short end, the plumbing cannot be approximate—it must be exact.

**Why this matters for middle-office readers:** When your desk's daily P&L shows "funding cost" or "carry," it reflects these money-market rates. When risk reports show overnight exposure or short-term rate sensitivity, they're measuring exposure to the instruments covered here. Understanding money markets is the foundation for understanding how the trading desk actually makes (or loses) money on funding.

Prerequisites: [Chapter 1](chapters/chapter_01_market_quoting_calendars_cashflow_plumbing.md), [Chapter 2](chapters/chapter_02_time_value_discount_factors_replication.md), [Chapter 3](chapters/chapter_03_zero_forward_par_rates_triangle.md)

Follow-on: [Chapter 5](chapters/chapter_05_fixed_rate_bond_pricing.md), [Chapter 9](chapters/chapter_09_repo_funding_engine.md), [Chapter 11](chapters/chapter_11_dv01_pv01_definitions_computation.md), [Chapter 17](chapters/chapter_17_curve_construction_bootstrapping_interpolation.md), [Chapter 24](chapters/chapter_24_stir_futures_convexity_adjustments.md)

This chapter covers:

1. **Core concepts**: Discount factors, accrual factors, simple rates, and the forward-rate identity that ties them together
2. **The credit hierarchy**: Why Fed Funds, T-bills, repo, and unsecured rates differ—and what the LIBOR-to-SOFR transition changed permanently
3. **Rate determination**: The critical distinction between rates fixed "in advance" (LIBOR) versus "in arrears" (SOFR)
4. **Instrument mechanics**: What deposits, bills, FRAs, and Fed Funds futures actually quote and how each pins down a curve point
5. **Fed probability extraction**: How traders calculate implied rate hike probabilities from Fed Funds futures
6. **Year-end dynamics**: Why funding rates spike at quarter-end and how traders position for the "turn"
7. **Mathematical derivations**: Step-by-step with unit checks and sanity tests
8. **Bootstrapping**: How to build the short end from market quotes
9. **Worked examples**: Ten detailed numeric calculations

We work primarily in a **single-curve setting** for pedagogy. The multi-curve reality of post-crisis practice—separate discounting and forwarding curves—is previewed but deferred to Part IV (Chapters 17-22).

## Learning Objectives
- Translate a money-market quote (deposit, bill discount yield, FRA, Fed Funds futures) into cashflows or implied rates and then into discount factors.
- Use the simple-forward identity to move between discount factors and forward rates with an explicit year-fraction \(\tau(\cdot,\cdot)\).
- Explain the operational meaning of “fixed in advance” vs “fixed in arrears” and why it matters for cash management.
- Compute and interpret a 1bp PV sensitivity with an explicit bump object, units, and sign convention.
- Spot and avoid the most common front-end breaks (day count, yield convention, averaging/compounding, and “what is being bumped?”).

---

## 4.1 Core Concepts

### 4.1.0 The Money Market Ecosystem

Before diving into the math, it helps to understand the "Credit Hierarchy" of the short end. Money markets are not a monolith; they are strata of trust.

> **Visualizing the Credit Pyramid**
>
> *   **Tier 1 (Policy / Administered)**: Central bank administered rates and facilities that anchor overnight funding.
> *   **Tier 2 (Overnight Unsecured)**: Overnight unsecured interbank funding (e.g., effective fed funds).
> *   **Tier 3 (Overnight Secured)**: Treasury general-collateral (GC) repo; transaction-based indices like SOFR reflect this secured funding layer.
> *   **Tier 4 (Sovereign Bills)**: T-bills and other government money-market securities—high quality, but still subject to liquidity and “specialness” dynamics.
> *   **Tier 5 (Unsecured Term Credit)**: Term bank/corporate funding (CDs/CP; historically LIBOR-style benchmarks) embedding credit and liquidity premia.
>
> **The Lesson of 2008:** In normal times, these rates tend to move together. In stress, the stack “pulls apart”: unsecured credit layers cheapen (rates rise), safe-haven layers richen (yields fall), and secured funding can dislocate due to balance-sheet and collateral constraints.

### 4.1.1 Credit-Sensitive vs Risk-Free Reference Rates: The Post-LIBOR Reality

The 2008 crisis and subsequent LIBOR transition revealed a fundamental truth about money market rates: **not all "short-term rates" are the same**. The distinction between credit-sensitive and risk-free rates now shapes how curves are built, swaps are priced, and corporate hedging programs are designed.

One useful mental split:

- **(Nearly) risk-free / RFR-style rates** are designed to reflect secured or policy-anchored overnight funding and to contain little bank credit premium (e.g., SOFR is based on secured Treasury repo transactions).
- **Credit-sensitive term rates** embed bank credit and liquidity premia (historically LIBOR).

Hull emphasizes this distinction: new overnight reference rates are intended to be (nearly) risk-free, while LIBOR incorporated a time-varying bank credit spread. Hull also notes a practical implication: when banks lend to customers at “SOFR + spread”, changes in that spread are not hedged by a plain SOFR leg.

> **Desk Reality:** Hedging a credit-sensitive funding exposure with an RFR hedge leaves residual spread risk.
> **Common break:** A funding P&L line (often “index + spread”) doesn’t match the P&L from a hedge that references only the index.
> **What to check:** Identify the exact reference index (SOFR vs EFFR vs term), the day count, and whether you need an explicit spread/basis hedge.

The practical lesson for curve work: always name the index you are discounting on and the index you are projecting (they may differ), and be explicit about the basis you are ignoring in a single-curve pedagogical setup.

### 4.1.2 The Discount Factor $P(0,T)$

The discount factor $P(0,T)$ is the time-0 value of receiving one unit of currency at time $T$. It is the fundamental building block of fixed-income pricing.

**Formal definition:** $P(0,T)$ is the price today of a zero-coupon bond that pays 1 at maturity $T$ (i.e., the present value of 1 paid at $T$).

**Intuition:** Think of $P(0,T)$ as the "present-value weight" applied to any cashflow at time $T$. At the short end of the curve, discount factors are close to 1—a payment due in 30 days is worth almost its face value—but even small deviations matter because short-dated instruments have tight bid-ask spreads and high trading frequency.

**Practice:** Front-end discount factors are the first "curve nodes" that anchor PVs of very short cashflows and the earliest forward rates used in floating-rate products.

### 4.1.3 The Year Fraction $\tau(T,S)$ and Day Count

A day-count convention converts dates into a **year fraction**. We write the year fraction between dates $T$ and $S$ as $\tau(T,S)$ (units: years). For a simple annualized rate $r$ applied to notional $N$ over $[T,S]$:

$$\boxed{\text{Interest} = N \cdot r \cdot \tau(T,S)}$$

**Why this matters at the short end:** When maturities are measured in days and weeks, a one-day error in $\tau$ can be comparable to a basis-point error in $r$.

**Convention variability:** Day-count conventions differ by market and instrument. In USD money markets, a common convention is **ACT/360** (so $d$ calendar days corresponds to $\tau=d/360$).

### 4.1.4 Simple (Money-Market) Rates

Money-market instruments usually quote **simple annualized rates** over their accrual period. A notional $N$ accruing at a simple rate $r$ over $[0,T]$ pays:

$$\boxed{N\cdot(1+r\cdot\tau(0,T)) \text{ at } T}$$

**From a deposit quote to a discount factor:** In a simple single-period setting, the par condition implies:

$$\boxed{P(0,T) = \frac{1}{1 + r \cdot \tau(0,T)}}$$

This formula captures the essence of short-end curve construction: a deposit quote is "directly a discount factor" once you apply the correct day-count convention.

### 4.1.5 The Forward Rate Identity

For any future period $[T_1,T_2]$, the simply-compounded forward rate $f(t;T_1,T_2)$ satisfies:

$$\boxed{1 + \tau(T_1,T_2)\cdot f(t;T_1,T_2) = \frac{P(t,T_1)}{P(t,T_2)}}$$

**Intuition:** The forward rate is the breakeven simple interest rate over $[T_1,T_2]$ implied by the discount curve.

**Solving for the forward rate:**

$$\boxed{f(0;T_1,T_2) = \frac{\frac{P(0,T_1)}{P(0,T_2)} - 1}{\tau(T_1,T_2)}}$$

### 4.1.6 What "Par" Means and How It Pins Down a Curve Point

An instrument is **at par** if its present value equals zero (for a derivative) or equals the notional invested (for a deposit or bill purchase at its quoted price).

For a deposit, "par" means the discounted maturity payoff equals the amount invested:

$$1 = P(0,T) \cdot (1 + r \cdot \tau(0,T))$$

This equation is the key to bootstrapping: each par quote lets you solve for one unknown discount factor.

---

## 4.2 Rate Determination: "In Advance" vs "In Arrears"

The LIBOR-to-SOFR transition introduced a fundamental change in how floating rates are determined. Understanding this distinction is essential for pricing, hedging, and operational cash management.

### 4.2.1 Rates Fixed "In Advance" (The LIBOR World)

Under the traditional LIBOR convention, the floating rate for an accrual period was **known at the start of that period**.

Hull explains: "LIBOR rates are forward looking and published at the beginning of periods to which they apply."

**Timeline example (3-month LIBOR):**
- **January 1**: 3M LIBOR fixes at 5.00%
- **January 1 – March 31**: Accrual period (rate is known = 5.00%)
- **March 31**: Payment calculated and made based on the 5.00% rate

**Operational advantage:** Cash managers knew exactly what payment was due well in advance. Funding could be arranged and hedging was straightforward.

### 4.2.2 Rates Fixed "In Arrears" (The SOFR World)

Under SOFR and other overnight risk-free rates, the floating rate is determined by **compounding daily observations over the accrual period**. The rate is not known until the period ends.

Hull emphasizes that the new overnight reference rates are **backward looking**: the rate for an accrual period is computed from overnight rates observed during the period, so it is only known at (or near) period end.

**Timeline example (3-month compounded SOFR):**
- **January 1**: Accrual period begins. Rate is **unknown**.
- **January 1 – March 31**: Daily SOFR rates are observed and compounded
- **Late March**: Rate becomes known only near period end (exact timing depends on the contract’s observation/payment conventions)
- **March 31**: Payment calculated and made (often with conventions to ensure operational notice)

**The compounding formula:** Hull provides the formula for computing the compounded rate over $n$ business days:

$$\boxed{\text{Compounded Rate} = \left[\prod_{i=1}^{n}\left(1 + r_i \cdot \frac{d_i}{360}\right) - 1\right] \times \frac{360}{D}}$$

where $r_i$ is the overnight rate on day $i$, $d_i$ is the number of calendar days that rate applies (typically 1, but 3 over weekends), and $D$ is the total calendar days in the period.

> **Desk Reality: Operational Challenges of "In Arrears"**
>
> **The cash management problem:** Under LIBOR, a corporate treasurer knew on January 1 that the March 31 payment would be exactly $X. Under compounded SOFR, the rate for the accrual period is not known until the end of the period, after the relevant overnight rates have been observed.
>
> **What to check:** Treat the observation/payment convention as part of the trade specification, and reproduce the compounded factor from the exact daily fixings your system uses.

### 4.2.3 Geometric vs Arithmetic Averaging

A subtle but important distinction exists in how overnight rates are combined:

**Geometric (compounded) averaging:** Used for term SOFR calculations and most SOFR-linked products. Each day's rate is compounded: $(1 + r_1)(1 + r_2)...(1 + r_n) - 1$. This is economically correct—it reflects actual reinvestment.

**Arithmetic averaging:** Used for Fed Funds futures settlement. The contract settles based on the simple arithmetic average of daily effective Fed Funds rates over the month.

Hull notes that an average overnight rate over a period is computed by compounding the daily overnight rates (a geometric average).

This distinction matters for basis calculations between Fed Funds futures and compounded SOFR.

---

## 4.3 Instruments at the Short End: What Is Quoted and What It Pins Down

### 4.3.1 Deposits and Money-Market Instruments

#### The Simple Interest Deposit Quote

A money-market deposit quotes a simple annualized rate $r$ for a maturity $T$, together with a day-count convention that determines the year fraction $\tau(0,T)$.

**What the quote pins down:** The discount factor $P(0,T)$ via the par condition:

$$1 = P(0,T) \cdot (1 + r \cdot \tau(0,T))$$

Solving:

$$\boxed{P(0,T) = \frac{1}{1 + r \cdot \tau(0,T)}}$$

Hull notes that USD money-market instruments typically use Actual/360 day count: the year fraction is actual days divided by 360.

#### Accrual and Settlement Conventions

Several operational conventions affect how deposit quotes translate to discount factors:

1. **Day count:** U.S. money markets use Actual/360; other markets may use Actual/365.
2. **Spot lag:** Many interbank rates have a settlement lag (e.g., T+2 for legacy USD LIBOR, T+0 for Fed Funds, T+1 for some Bills).
3. **Business day adjustments:** Modified following is common for date roll conventions.

For pedagogical examples in this chapter, we ignore spot lags and assume year fractions are given directly.

### 4.3.2 Treasury Bills: Discount Instruments

> **Repo 101: The Plumbing of Wall Street**
>
> A **Repurchase Agreement (Repo)** is legally a *sale* and a *repurchase*, but economically a **collateralized loan**.
> *   **Leg 1**: I sell you a Treasury bond for $100 today.
> *   **Leg 2**: I agree to buy it back tomorrow for $100.01.
> *   **Economics**: I borrowed $100 from you overnight at a 1bp interest rate, using the bond as collateral.
>
> Repo rates (like SOFR) form the bedrock of the modern "Tier 3" secured funding market. Full coverage of repo mechanics is in Chapter 9.

#### Bill Price Quoting vs. Discount Yield Quoting

Hull notes that some money-market instruments (including US T-bills) are quoted using a discount rate based on face value rather than on the price paid.

> **Visualizer: Discount Yield vs. Investment Yield**
>
> Why do we have two yields? Because we are comparing "Apples to Oranges".
>
> *   **Discount Yield (The "Discount")**: "I take 2% off the top."
>     *   Base: **Face Value** ($F$).
>     *   Math: $\frac{D}{F}$.
>     *   Example: Pay 98, Get 100. Yield = 2/100 = 2%.
> *   **Investment Yield (The "Return")**: "I add 2% to the bottom."
>     *   Base: **Price Paid** ($P$).
>     *   Math: $\frac{D}{P}$.
>     *   Example: Pay 98, Get 100. Return = 2/98 = 2.04%.
>
> **Rule (mechanical):** If $Y<100$ is the cash price per \$100 face, then $(100-Y)/Y > (100-Y)/100$, so a price‑denominator yield is higher than a face‑denominator “discount yield”.

A US Treasury bill with 91 days to maturity quoted at 8 means “8% of face value per 360 days.” Hull illustrates: for \$100 face, interest over 91 days is \$100 × 0.08 × 91/360 = \$2.0222, so the cash price is \$97.9778 per \$100 face.

Let $Y$ be the **cash price per \$100 face**, let $d$ be the remaining life in calendar days, and let $q_{\text{disc}}$ be the quoted **bank discount rate** (percent per year, ACT/360, applied to face value). Hull gives the relationship:

$$P = \frac{360}{d}(100 - Y)$$

where $P$ is his symbol for the quoted discount rate and $Y$ is the cash price. In this book we write the quote as $q_{\text{disc}}$ (so $q_{\text{disc}}=P$):

$$\boxed{q_{\text{disc}} = \frac{360}{d}(100 - Y)}$$

Solving for the cash price:

$$\boxed{Y = 100 - q_{\text{disc}}\frac{d}{360}}$$

#### From Bill Price to Discount Factor

Once you have the cash price per \$100 face $Y$:

$$P(0,T) = \frac{Y}{100}$$

#### Caution: Multiple Yield Conventions Exist

An **investment-style yield** (returns on price paid) differs from the bank discount quote. Hull’s example highlights the core point: a return computed on the **price paid** is higher than the discount computed on **face value** because $Y<100$.

One commonly used annualization (often discussed as a “bond‑equivalent” style conversion for bills) is to annualize the holding-period return on a 365-day basis (Musiela):

$$y_{\text{ask}} = \left(\frac{100 - Y}{Y}\right) \times \frac{365}{d}$$

Note the differences:
- Denominator: price (not face)
- Annualization: 365 (not 360)

The banker's discount yield is a convenient quoting convention, **not** an internal rate of return.

> **Pitfall — Discount yield is not an IRR:** A T-bill “discount yield” annualizes \((F-\text{Price})/F\) on a 360-day basis, while investment yields annualize returns on the price paid.
> **Why it matters:** Comparing discount yields directly to deposit/OIS rates (or using them as an IRR) leads to wrong relative-value and risk numbers.
> **Quick check:** Convert quote → cash price first; then compute the simple holding-period return \((F/\text{Price}-1)/\tau(0,T)\) using an explicit day count.

### 4.3.3 Forward Rate Agreements (FRAs)

#### FRA as a Forward-Starting Deposit

Hull describes an FRA as a contract that exchanges a fixed rate for a reference rate observed in the future on a specified notional, typically settled net without exchanging principal.

An FRA references the future floating rate for a period $[T_1, T_2]$ and exchanges (in net form) the difference between floating and a fixed contract rate $K$. Let $\tau(T_1,T_2)$ be the year fraction for the accrual period under the contract’s day count.

> **Analogy: "Pre-Ordering" Your Loan**
>
> An FRA is like ordering a pizza for next month at today's fixed price.
> *   **Lock-In**: You agree to pay 5% for a 3-month loan starting in June.
> *   **Scenario A (Rates Rise to 7%)**: The market loan costs 7%. You are happy! The FRA seller pays you the difference (2%). Net cost = 5%.
> *   **Scenario B (Rates Fall to 3%)**: The market loan costs 3%. You are sad. You must pay the FRA seller the difference (2%). Net cost = 5%.
>
> **Result**: No matter where rates go, your net effective rate is locked at 5%.

#### FRA Value in Terms of Discount Factors

The value of an FRA at time $t \leq T_1$ for unit notional can be written:

$$V_{\text{FRA}}(t) = P(t,T_1) - P(t,T_2) - K \cdot \tau(T_1,T_2)\cdot P(t,T_2)$$

This can be rewritten as:

$$V_{\text{FRA}}(t) = \tau(T_1,T_2) \cdot P(t,T_2) \cdot \left(L(t;T_1,T_2) - K\right)$$

where $L(t;T_1,T_2)$ is the (simply-compounded) forward rate implied by the discount curve.

Hull notes that FRAs can be valued using the forward rate for the underlying period implied by the curve, discounted appropriately.

#### The Par FRA Rate Equals the Forward Rate

Setting $V_{\text{FRA}}(t) = 0$ and solving for the fixed rate $K$:

$$\boxed{K^* = L(t;T_1,T_2) = \frac{1}{\tau(T_1,T_2)}\left(\frac{P(t,T_1)}{P(t,T_2)} - 1\right)}$$

The par FRA rate equals the implied forward rate—a fundamental result that makes FRAs direct constraints on the forward curve. Hull notes that the FRA has zero value when the fixed rate equals the relevant forward rate.

#### FRA Settlement Payoff

At settlement (time $T_1$), the net payment (paid at $T_1$) is:

$$\boxed{V_{\text{FRA}}(T_1) = \frac{\tau(T_1,T_2) \cdot (L(T_1;T_1,T_2) - K)}{1 + \tau(T_1,T_2) \cdot L(T_1;T_1,T_2)}}$$

The denominator discounts the net interest difference from $T_2$ back to $T_1$ at the realized floating rate for the accrual period.

### 4.3.4 Fed Funds Futures: Contract Structure

Tuckman describes fed funds futures as designed to hedge a \$5,000,000 30-day fed funds deposit; the final settlement is based on the average effective fed funds rate over the contract month.

**Key contract features:**
- **Notional:** $5,000,000
- **Settlement:** Cash-settled based on the arithmetic average of daily effective Fed Funds rates during the contract month
- **Quote:** $100 - \text{(average rate in percent)}$

**Averaging convention:** Unlike SOFR (geometric compounding), Fed Funds futures use **arithmetic averaging**:

$$\text{Average Rate} = \frac{1}{N}\sum_{i=1}^{N} r_i$$

where $N$ is the number of calendar days in the month and $r_i$ is the effective Fed Funds rate on day $i$.

> **Desk Reality:** Tuckman describes the fed funds futures contract as a hedge to a \(\$5{,}000{,}000\) 30-day deposit in fed funds, and the final settlement price is set to 100 minus 100 times the average effective fed funds rate over the month.
> **Common break:** In fed funds futures, changing the rate of the \(\$5{,}000{,}000\) 30-day underlying by 1 bp changes the interest payment by \(\$41.67\) (= \(\$5{,}000{,}000 \times (.0001 \times 30)/360\)).
> **What to check:** Tuckman writes that hedging a \(\$100{,}000{,}000\) investment over December requires \(20\times(31/30)\) or 21 fed funds futures contracts.

### 4.3.5 The "Staircase" Curve: Fed Funds at FOMC Meetings

Tuckman notes that the Fed usually changes the fed funds target at the conclusion of scheduled FOMC meetings (with occasional exceptions). A convenient mental model for expected fed funds rates is therefore a stepwise curve: constant between meetings and jumping at meeting dates.

This creates a **staircase pattern** in expected Fed Funds rates:
- Rates are approximately constant between FOMC meetings
- Rates "step" up or down at each FOMC meeting date (if a change is expected)

### 4.3.6 Fed Probability Extraction from Futures Prices

One of the most practical applications of Fed Funds futures is extracting the market-implied probability of Fed rate changes. Tuckman provides a detailed methodology.

**The setup:**
- An FOMC meeting occurs during the futures contract month
- Before the meeting: rate is $r_0$ (current target)
- After the meeting: rate is either $r_0$ (no change) or $r_1$ (new target)
- Meeting occurs on day $m$ of month with $N$ total days

**The futures price reflects the weighted average:**

$$\text{Implied Average} = \frac{m-1}{N} \times r_0 + \frac{N-m+1}{N} \times r_{\text{expected post-meeting}}$$

**Solving for the expected post-meeting rate:**

$$r_{\text{expected}} = \frac{N \times r_{\text{implied}} - (m-1) \times r_0}{N - m + 1}$$

**Extracting probability:** If the only two outcomes are "no change" (rate stays at $r_0$) or "change to $r_1$":

$$\boxed{p = \frac{r_{\text{expected}} - r_0}{r_1 - r_0}}$$

where $p$ is the probability of a rate change.

> **Desk Reality: "The Market is Pricing 3 Cuts This Year"**
>
> When you hear traders or CNBC say "the market is pricing ~75bp of cuts over the next year," they're using exactly this methodology—applied sequentially across multiple FOMC meetings.
>
> **The calculation chain:**
> 1. Use near-month futures to extract probability of next meeting's action
> 2. Use further months to extract conditional probabilities of subsequent meetings
> 3. Sum expected rate changes across all meetings
>
> **Limitations:**
> - Futures prices also embed risk premia (not pure expectations)
> - The probability extraction assumes only two outcomes; reality may include 50bp moves
> - Meeting date placement within the month affects precision

### 4.3.7 STIR Futures: Forward-Rate Instruments (Preview Only)

#### Quote Convention

Andersen & Piterbarg frame Eurodollar (and similarly SOFR) futures as forward-rate instruments entered at zero upfront cost, with payoff tied to the realized floating rate over the underlying period relative to the rate implied at inception.

The quoted price is typically:

$$Q(t;T) = 100 \times (1 - \text{futures rate})$$

For example, a quote of 94.67 implies a futures rate of 5.33%.

**SOFR futures distinction:** Unlike legacy Eurodollar futures (which referenced a single 3M LIBOR fixing), SOFR futures reference the compounded overnight rate over the contract period. This is an average of daily rates, not a single term rate fixing.

#### Futures vs. Forwards: The Convexity Adjustment

Because futures are marked to market daily while FRAs settle at maturity, the futures-implied rate generally differs from the corresponding forward rate. Andersen & Piterbarg explain that daily settlement creates a correlation between margin cashflows and funding/reinvestment rates, so futures rates generally differ from forward rates; the futures rate tends to be above the corresponding forward rate.

The mechanism is as follows: under rising interest rates, the futures holder must make margin payments at precisely the moment when borrowing costs are highest. Conversely, when rates fall, received margin is reinvested at lower rates. This systematic disadvantage means futures rates must exceed forward rates to compensate.

Hull provides similar intuition: daily settlement creates a convexity adjustment, and the forward and futures quotes differ as a result.

**Magnitude:** Hull notes that the convexity adjustment increases as the life of the contract increases.

The full convexity adjustment derivation is covered in Chapter 24.

---

## 4.4 Seasonal Funding Dynamics: The Year-End "Turn"

### 4.4.1 Why Special Dates Dislocate (The "Turn")

In this chapter, we use the practitioner term **turn** for the implied overnight rate over a stub spanning a special date (for example, the overnight rate spanning December 31).

Tuckman's Figure 17.2 shows that the fed funds effective rate and the fed funds target rate can be very far apart, reminding you that the very front end is not guaranteed to be smooth.

> **Practitioner Note: What to internalize**
>
> - Treat the turn as a **stub forward** implied by discount factors across the stub.
> - If you are quoted a turn (e.g., “Dec/Jan”), translate it into a local constraint on the nearest discount factors and forwards.

### 4.4.2 Pricing the Turn

**Extracting the turn rate:** If you have discount factors $P(0, \text{Dec 30})$ and $P(0, \text{Jan 2})$, you can extract the implied overnight rate spanning the turn:

$$r_{\text{turn}} = \left(\frac{P(0, \text{Dec 30})}{P(0, \text{Jan 2})} - 1\right)\times \frac{360}{d}$$

where $d$ is the number of calendar days (typically 3 for a weekend).

**Trading the turn:**
- **Long turn:** Lend over the stub if you think the realized rate will be higher than what the curve implies
- **Short turn:** Borrow over the stub if you think the realized rate will be lower than what the curve implies

### 4.4.3 Quarter-End Effects

The same math applies to other special-date stubs (month-ends, quarter-ends, etc.). Identify the stub dates and compute the implied forward from the discount factors.

---

## 4.5 Mathematical Derivations

### 4.5.1 Deposit Quote → Discount Factor

**Setup:**
- Simple interest over $[0,T]$
- Unit principal invested at $t = 0$
- No default; single-curve framework

**Cashflow:** The deposit payoff at $T$ is $1 + r \cdot \tau(0,T)$.

**Par condition:** The present value at inception equals 1:

$$1 = P(0,T) \cdot (1 + r \cdot \tau(0,T))$$

**Solve for the discount factor:**

$$\boxed{P(0,T) = \frac{1}{1 + r \cdot \tau(0,T)}}$$

**Unit check:** Rate $r$ has units "per year"; $\tau$ has units "years"; so $r \cdot \tau$ is dimensionless, and $P$ is dimensionless. ✓

**Sanity checks:**
- If $r$ increases, the denominator increases, so $P(0,T)$ decreases. ✓
- As $T \to 0$, $\tau(0,T) \to 0$, so $P(0,T) \to 1$. ✓

### 4.5.2 Forward Rate from Discount Factors

From the forward-rate identity:

$$1 + \tau(T_1,T_2) \cdot F(0;T_1,T_2) = \frac{P(0,T_1)}{P(0,T_2)}$$

Solving for the forward rate:

$$\boxed{F(0;T_1,T_2) = \frac{\frac{P(0,T_1)}{P(0,T_2)} - 1}{\tau(T_1,T_2)}}$$

**Unit check:** Numerator is dimensionless; dividing by $\tau$ (years) gives "per year." ✓

**Sanity check:** If $P(0,T_2)$ is smaller (more discounting to $T_2$), the forward rate is larger. ✓

### 4.5.3 FRA Par Rate Equals the Forward Rate

Let the FRA accrual period be $[T_1,T_2]$ with year fraction $\tau(T_1,T_2)$.

From the FRA value expression:

$$V_{\text{FRA}}(t) = P(t,T_1) - P(t,T_2) - K \cdot \tau(T_1,T_2)\cdot P(t,T_2)$$

Setting $V_{\text{FRA}}(t) = 0$:

$$P(t,T_1) = P(t,T_2) \cdot (1 + K\cdot \tau(T_1,T_2))$$

Solving for $K$:

$$K = \frac{1}{\tau(T_1,T_2)}\left(\frac{P(t,T_1)}{P(t,T_2)} - 1\right)$$

But by the forward-rate identity:

$$L(t;T_1,T_2) = \frac{1}{\tau(T_1,T_2)}\left(\frac{P(t,T_1)}{P(t,T_2)} - 1\right)$$

Therefore:

$$\boxed{K^* = L(t;T_1,T_2) = F(t;T_1,T_2)}$$

### 4.5.4 FRA Time-0 Present Value

For notional $N$ and fixed rate $K$:

$$\boxed{V_{\text{FRA}}(0) = N \cdot \tau(T_1,T_2) \cdot P(0,T_2) \cdot (F - K)}$$

where $F = L(0;T_1,T_2)$ is the forward rate.

**Sanity checks:**
- If $K = K^*$ (par), then $V_{\text{FRA}}(0) = 0$. ✓
- For the fixed-rate payer, value increases when the forward rate increases. ✓

---

## 4.6 Bootstrapping the Short End

### 4.6.1 The Sequential Logic

Curve construction is an inverse problem: given prices of traded instruments, recover the discount factors. Andersen notes that only a finite set of instruments is quoted, so constructing a continuous curve inevitably requires bootstrapping plus interpolation.

**The short-end bootstrap:**

**Step 1: Initialize.** $P(0,0) = 1$.

**Step 2: Use deposits (or bills) for the earliest nodes.** For each maturity $T$ with deposit quote $r(0,T)$:

$$P(0,T) = \frac{1}{1 + r(0,T) \cdot \tau(0,T)}$$

For bills quoted via discount yield, first convert quote → price, then $P(0,T) = \text{price}/\text{face}$.

**Step 3: Use FRAs to extend further.** If $P(0,T_1)$ is known and you have a par FRA quote $K$ for $[T_1,T_2]$, then:

$$\boxed{P(0,T_2) = \frac{P(0,T_1)}{1 + K \cdot \tau(T_1,T_2)}}$$

This follows directly from the forward-rate identity.

**Step 4: Interpolate between nodes.** Between instrument maturities, choose an interpolation scheme (on zero rates, log discount factors, or forward rates). This is a modeling choice with implications for smoothness and locality of perturbations.

### 4.6.2 Single-Curve vs. Multi-Curve (Preview)

Andersen notes that pre-2007 practice commonly used a single LIBOR curve; after the crisis, curve construction often requires multiple curves (discounting vs forwarding). For simplicity, this chapter proceeds with the single-curve assumption.

**The economic source of the basis:** Credit risk differences between overnight secured rates (SOFR/OIS) and term unsecured rates (historical LIBOR) create a spread. Even after LIBOR cessation, different tenors of SOFR-based rates can exhibit basis due to term premium and liquidity effects.

In USD rates markets, overnight indexed swap (OIS) curves referencing an overnight index (e.g., effective fed funds or SOFR, depending on the contract and collateralization) are often treated as the closest traded proxy for “risk‑free” discounting. In a multi-curve setup, one curve provides discount factors (the discounting/collateral curve) while separate curves provide forwards for the floating index being projected.

This chapter remains single-curve to explain the mechanics. **Full multi-curve construction methodology is covered in Part IV, particularly Chapters 18-20.**

---

## 4.7 Rate Sensitivity at the Short End

### 4.7.1 Sensitivity of Discount Factor to Rate

For a deposit-implied discount factor:

$$P(0,T) = \frac{1}{1 + r \cdot \tau}$$

The sensitivity to the quoted rate is:

$$\frac{\partial P}{\partial r} = -\frac{\tau}{(1 + r \cdot \tau)^2}$$

**Interpretation:** The short-end discount factor is most sensitive to (i) larger accrual fractions and (ii) lower rate levels. Even small $\tau$ matters because front-end PVs are dominated by short-dated cashflows.

**A concrete “01” (bump object + units + sign):** For a deterministic cashflow of notional $N$ paid at $T$, $PV = N\cdot P(0,T)$. Define the **local** DV01 to the simple quote $r$ used in $P(0,T)=1/(1+r\tau)$ (with $\tau:=\tau(0,T)$) as:
$$DV01 := PV(r-1\text{bp})-PV(r)$$
Using $P(0,T)=1/(1+r\tau)$ and $1\text{bp}=10^{-4}$, a first-order approximation is:
$$DV01 \approx N\cdot \frac{\tau}{(1+r\tau)^2}\cdot 10^{-4}\quad\text{(currency per 1bp; positive for long PV risk).}$$

### 4.7.2 Locality of Short-End Perturbations

A key property of sequential bootstrapping: changing a short-dated quote primarily affects the nearest nodes and the forwards that depend on them. Andersen describes "bucket exposure" analysis where bumping a specific forward-rate segment reveals the locality of curve shocks.

This locality intuition underlies key-rate DV01 analysis, covered in Chapter 14.

---

## 4.8 Worked Examples

### Numerical Conventions (Examples 1–10)

- **Day count:** Actual/360
- **Calendar simplification:** 1M = 30 days, 3M = 90 days, 6M = 180 days, 12M = 360 days
- **Rates:** Simple annualized money-market rates

---

### Example 1: Deposit → Discount Factor

**Given:**
- Overnight (1 day): $r_{\text{ON}} = 4.75\%$
- 1M (30 days): $r_{1M} = 5.00\%$
- 3M (90 days): $r_{3M} = 5.20\%$

**Accrual factors (ACT/360):**

| Tenor | Days | $\tau(0,T)$ |
|-------|------|----------|
| ON | 1 | 1/360 = 0.00277778 |
| 1M | 30 | 30/360 = 0.08333333 |
| 3M | 90 | 90/360 = 0.25 |

**Discount factors:**

**Overnight:**
$$P(0,\text{ON}) = \frac{1}{1 + 0.0475 \times 0.00277778} = \frac{1}{1.00013194} = 0.99986807$$

**1M:**
$$P(0,1M) = \frac{1}{1 + 0.0500 \times 0.08333333} = \frac{1}{1.00416667} = 0.99585062$$

**3M:**
$$P(0,3M) = \frac{1}{1 + 0.0520 \times 0.25} = \frac{1}{1.013} = 0.98716683$$

**Sanity check:** Higher rate or longer accrual → lower discount factor. ✓

---

### Example 2: Two Deposits → Implied Forward

**Compute the 1M–3M forward rate.**

Using:
$$F(0;T_1,T_2) = \frac{\frac{P(0,T_1)}{P(0,T_2)} - 1}{\tau(T_1,T_2)}$$

With:
- $P(0,1M) = 0.99585062$
- $P(0,3M) = 0.98716683$
- $\tau(1M,3M) = 60/360 = 0.16666667$

**Step 1:** Compute the ratio:
$$\frac{P(0,1M)}{P(0,3M)} = \frac{0.99585062}{0.98716683} = 1.00879668$$

**Step 2:** Compute the forward:
$$F(0;1M,3M) = \frac{1.00879668 - 1}{0.16666667} = \frac{0.00879668}{0.16666667} = 0.05278008$$

$$\boxed{F(0;1M,3M) \approx 5.2780\%}$$

---

### Example 3: Bill Discount Quote → Price/DF

**Example Title**: 91-day T-bill (discount quote → price → DF → DV01)

**Context**
- What is being priced/measured? A single-payment T-bill priced from a bank discount quote.
- Why it matters on the desk: This is a front-end curve anchor; you also need a clear “01” to the *quote object* to sanity-check risk reports.

**Timeline (Make Dates Concrete)**
- Trade date: 2026-03-02
- Settlement date: 2026-03-02 (assumed; ignore spot lag / business-day adjustments)
- Payment date: 2026-06-01 (91 calendar days later)

**Inputs**
- Face value: \$100 per \$100 face (quote basis); position size: \$100,000,000 face
- Days to maturity: \(d=91\) (bank discount quotes annualize on ACT/360)
- Bank discount quote: \(q_{\text{disc}}=5.20\) (percent per year; ACT/360; applied to face)

**Outputs (What You Produce)**
- Cash price per \$100 face
- Discount factor \(P(0,T)\)
- Risk metric: \(DV01 := PV(q_{\text{disc}}-1\text{bp})-PV(q_{\text{disc}})\) (units: USD per 1bp; bump object = bank discount quote at this maturity)

**Step-by-step**
1. Translate quote to cash price (per \$100 face):
   $$Y=100-q_{\text{disc}}\frac{d}{360}=100-5.20\frac{91}{360}=98.6856$$
2. Convert price to discount factor:
   $$P(0,T)=\frac{Y}{100}=0.9868556$$
3. Compute DV01 (quote bump, rates down 1bp):
   Since $q_{\text{disc}}$ is quoted in percent, $1\text{bp}=0.01$ in this quote unit. Therefore:
   $$DV01=PV(q_{\text{disc}}-1\text{bp})-PV(q_{\text{disc}})=\frac{\$100{,}000{,}000}{100}\cdot \frac{d}{360}\cdot 0.01=\$2{,}527.78$$

**Cashflows (table)**
| Date | Cashflow | Explanation |
|---|---|---|
| 2026-03-02 | \(-\$98{,}685{,}556\) | Pay cash price for \$100mm face |
| 2026-06-01 | \(+\$100{,}000{,}000\) | Receive face at maturity |

**P&L / Risk Interpretation**
- For a long bill, a 1bp *drop* in the quoted bank discount quote increases PV by about \$2.5k per \$100mm face (under this bump definition).
- Do not compare \(q_{\text{disc}}\) directly to money-market yields without converting to a consistent yield basis.

**Sanity Checks**
- Units check: $q_{\text{disc}}$ is percent per year and $d/360$ is years, so $q_{\text{disc}}d/360$ is the percent discount off face over the period.
- Sign check: quote down → price up → positive DV01 for a long bill (book DV01 convention).
- Limit check: if $q_{\text{disc}}=0$, then $Y=100$.

**References**
- (Hull, *Options, Futures, and Other Derivatives*, “Price Quotations of U.S. Treasury Bills”)

---

### Example 4: FRA Par Rate from Discount Factors

**Add a 6M deposit:** $r_{6M} = 5.30\%$, $\tau(0,6M) = 0.5$

**Step 1:** Compute $P(0,6M)$:
$$P(0,6M) = \frac{1}{1 + 0.0530 \times 0.5} = \frac{1}{1.0265} = 0.97418400$$

**Step 2:** Compute the 3M–6M par FRA rate:
- $\tau(3M,6M) = 90/360 = 0.25$
- $P(0,3M) = 0.98716683$

$$F(0;3M,6M) = \frac{\frac{0.98716683}{0.97418400} - 1}{0.25} = \frac{1.01332688 - 1}{0.25} = \frac{0.01332688}{0.25}$$

$$\boxed{K^* = F(0;3M,6M) \approx 5.3308\%}$$

---

### Example 5: FRA PV with Non-Par Rate

**Given:**
- Notional $N = 100{,}000{,}000$
- Period: 3M to 6M, $\tau = 0.25$
- $P(0,6M) = 0.97418400$
- Forward: $F = 5.3308\%$
- Contract rate: $K = 5.4308\%$ (10 bp above par)

**PV formula:**
$$V_{\text{FRA}}(0) = N \cdot \tau \cdot P(0,6M) \cdot (F - K)$$

**Compute:**
- $F - K = 0.053308 - 0.054308 = -0.0010$
- $V_{\text{FRA}}(0) = 100{,}000{,}000 \times 0.25 \times 0.97418400 \times (-0.0010)$
- $= 25{,}000{,}000 \times 0.97418400 \times (-0.0010)$
- $= -24{,}354.60$

$$\boxed{V_{\text{FRA}}(0) \approx -\$24{,}355}$$

The fixed-rate payer (receive floating, pay fixed) has negative value when paying above the market forward.

---

### Example 6: Mini Bootstrap to 12M

**Quotes:**
| Tenor | Type | Quote | $\tau$ |
|-------|------|-------|----------|
| 1M | Deposit | 5.00% | 30/360 |
| 3M | Deposit | 5.20% | 90/360 |
| 6M | Deposit | 5.30% | 180/360 |
| 6x12 | FRA | 5.40% | 180/360 |

**Step 1:** Deposits → discount factors (from earlier examples):
- $P(0,1M) = 0.99585062$
- $P(0,3M) = 0.98716683$
- $P(0,6M) = 0.97418400$

**Step 2:** FRA 6x12 → solve for $P(0,12M)$:

$$P(0,12M) = \frac{P(0,6M)}{1 + K \cdot \tau(6M,12M)} = \frac{0.97418400}{1 + 0.054 \times 0.5} = \frac{0.97418400}{1.027} = 0.94857173$$

**Discount Factor Table:**

| Maturity | Days | $\tau(0,T)$ | Source | Quote | $P(0,T)$ |
|----------|------|---------------|--------|-------|----------|
| 0 | 0 | 0 | — | — | 1.00000000 |
| 1M | 30 | 0.083333 | Deposit | 5.00% | 0.99585062 |
| 3M | 90 | 0.250000 | Deposit | 5.20% | 0.98716683 |
| 6M | 180 | 0.500000 | Deposit | 5.30% | 0.97418400 |
| 12M | 360 | 1.000000 | FRA 6x12 | 5.40% | 0.94857173 |

---

### Example 7: Quote Bump Sensitivity

**Bump the 1M deposit by +1 bp:** $r'_{1M} = 5.01\%$

**Recompute $P(0,1M)$:**
$$P'(0,1M) = \frac{1}{1 + 0.0501 \times 0.08333333} = \frac{1}{1.004175} = 0.99584236$$

**Change in discount factor:**
$$\Delta P(0,1M) = 0.99584236 - 0.99585062 = -0.00000826$$

**Impact on 1M–3M forward (holding $P(0,3M)$ fixed):**

$$F'(0;1M,3M) = \frac{\frac{0.99584236}{0.98716683} - 1}{0.16666667} = \frac{0.00878831}{0.16666667} = 5.2730\%$$

**Summary:** A 1 bp bump in the 1M deposit lowered the 1M–3M forward by about 0.5 bp. Local quote changes have local (but not unit-for-unit) effects on forwards.

---

### Example 8: STIR Futures Preview

**Given:** Futures price $Q = 94.67$ for a 3M rate starting at 3M.

**Step 1:** Implied futures rate:
$$R^{\text{fut}} = 1 - \frac{Q}{100} = 1 - 0.9467 = 0.0533 = 5.33\%$$

**Step 2:** Compare to the forward rate:
From Example 4: $F(0;3M,6M) \approx 5.3308\%$

**Observation:** In this toy setup, the futures and forward rates are essentially the same (the difference is tiny at this horizon). In general, because futures are marked-to-market daily, futures rates tend to be slightly **above** the corresponding forward rates (a convexity adjustment), and the gap typically becomes more visible as the contract maturity increases.

---

### Example 9: Fed Funds Futures — Probability Extraction

**Scenario:** It is December 1. The FOMC meets on December 14. We want to extract the probability of a 25bp rate cut.

**Given:**
- Current Fed Funds target: 5.25%
- December Fed Funds futures price: 94.85 (implying average rate = 5.15%)
- December has 31 days
- Meeting on day 14

**Step 1:** Calculate implied post-meeting average rate

The December average of 5.15% reflects:
- Days 1-13 (13 days) at the current rate: 5.25%
- Days 14-31 (18 days) at the post-meeting rate: unknown

$$5.15\% = \frac{13}{31} \times 5.25\% + \frac{18}{31} \times r_{\text{post}}$$

Solving for $r_{\text{post}}$:
$$r_{\text{post}} = \frac{31 \times 5.15\% - 13 \times 5.25\%}{18} = \frac{159.65\% - 68.25\%}{18} = \frac{91.40\%}{18} = 5.078\%$$

**Step 2:** Calculate probability of 25bp cut

If the only outcomes are:
- No cut: rate stays at 5.25%
- 25bp cut: rate goes to 5.00%

$$p_{\text{cut}} = \frac{5.25\% - 5.078\%}{5.25\% - 5.00\%} = \frac{0.172\%}{0.25\%} = 68.8\%$$

$$\boxed{p_{\text{cut}} \approx 69\%}$$

**Interpretation:** The market is pricing approximately 69% probability of a 25bp cut at the December FOMC meeting.

---

### Example 10: Compounded SOFR Rate Calculation

**Given:** Five consecutive business days of SOFR fixings:

| Day | Date | SOFR Rate | Days Applied |
|-----|------|-----------|--------------|
| 1 | Mon | 5.30% | 1 |
| 2 | Tue | 5.31% | 1 |
| 3 | Wed | 5.32% | 1 |
| 4 | Thu | 5.31% | 1 |
| 5 | Fri | 5.30% | 3 (covers weekend) |

**Total calendar days:** 7

**Step 1:** Apply compounding formula

$$\text{Compounded} = \prod_{i=1}^{5}\left(1 + r_i \cdot \frac{d_i}{360}\right) - 1$$

Calculate each factor:
- Day 1: $1 + 0.0530 \times \frac{1}{360} = 1.000147222$
- Day 2: $1 + 0.0531 \times \frac{1}{360} = 1.000147500$
- Day 3: $1 + 0.0532 \times \frac{1}{360} = 1.000147778$
- Day 4: $1 + 0.0531 \times \frac{1}{360} = 1.000147500$
- Day 5: $1 + 0.0530 \times \frac{3}{360} = 1.000441667$

**Step 2:** Multiply factors

$$\text{Product} = 1.000147222 \times 1.000147500 \times 1.000147778 \times 1.000147500 \times 1.000441667$$
$$= 1.001031890$$

**Step 3:** Annualize

$$\text{Compounded Rate} = (1.001031890 - 1) \times \frac{360}{7} = 0.001031890 \times 51.4286$$

$$\boxed{\text{Compounded SOFR} = 5.307\%}$$

**Note:** The compounded rate (5.307%) is very close to the simple average (5.308%) for such a short period. The difference grows for longer periods.

---

## 4.9 Practical Notes

### 4.9.1 Quoting Conventions to Watch

**Simple vs. compounded:** Money-market rates are simple; mixing them with continuously compounded rates without conversion causes errors.

**Day count matters:** ACT/360 is standard for U.S. money markets, but ACT/365 applies in some jurisdictions. Hull emphasizes these vary "from country to country and from instrument to instrument."

**Bill conventions are not interchangeable:** Banker's discount yield uses face in the denominator and 360-day annualization. The ask yield (BEY) uses price and 365-day annualization. Neither is an IRR.

**Basis points vs. decimals:** $1\text{ bp} = 0.0001$ in decimal. Many front-end pricing errors are unit mistakes.

### 4.9.2 Common Implementation Pitfalls

| Pitfall | Consequence |
|---------|-------------|
| Wrong day count | Systematic accrual error |
| Mixing ACT/360 and ACT/365 | Inconsistent discount factors |
| Ignoring spot lags | Wrong settlement dates |
| Bootstrap ordering errors | Solving for wrong unknowns |
| Unit confusion (bp vs %) | Order-of-magnitude errors |
| Arithmetic vs geometric avg | Small but systematic bias |
| In-advance vs in-arrears mismatch | Payment timing errors |

### 4.9.3 Verification Tests

**Repricing test for deposits:** Verify $P(0,T) \cdot (1 + r \cdot \tau(0,T)) = 1$.

**Repricing test for FRAs:** Verify par FRA has PV ≈ 0.

**Sign sanity:** Higher rates → lower discount factors.

**Bound checks:** $P(0,T) > 0$, typically $P(0,T) \leq 1$ for non-negative rates.

---

## Summary

1. **Short-end curve construction** begins with instruments whose PV-to-par conditions map directly to discount factors.

2. **A simple deposit** quoted at rate $r$ with year fraction $\tau(0,T)$ implies $P(0,T) = 1/(1 + r\,\tau(0,T))$.

3. **Day count is essential:** USD money markets typically use Actual/360, so year fractions are actual days divided by 360.

4. **The forward-rate identity** $1 + \tau F = P(0,T_1)/P(0,T_2)$ ties discount factors to forward rates.

5. **An FRA is a forward-starting deposit** in net-settlement form.

6. **The par FRA rate equals the forward rate** implied by the discount curve.

7. **Bill discount yields** are quoting conventions, not return measures. Convert to price first, then to discount factor.

8. **STIR futures quotes** map to implied rates; daily settlement creates a convexity adjustment (full treatment in Chapter 24).

9. **Bootstrapping** is sequential: use each instrument to solve for one new unknown.

10. **Post-crisis practice** uses multiple curves (OIS for discounting, term rates for projection). Full treatment in Part IV.

11. **SOFR vs LIBOR:** SOFR is a secured, risk-free overnight rate determined in arrears. LIBOR was an unsecured term rate determined in advance. This distinction affects hedging and operational cash management.

12. **Fed Funds futures** allow extraction of market-implied probabilities of Fed rate changes—a core rates desk skill.

13. **Year-end “turn”:** Special-date stub forwards can dislocate around year-end; treat the turn as a local stub forward implied by nearby discount factors.

---

## Key Concepts

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| Discount factor $P(0,T)$ | Price today of $1 at time $T$ | Fundamental pricing primitive |
| Year fraction $\tau(T_1,T_2)$ | Year fraction under day-count convention | Converts rates to cash interest amounts |
| Simple rate | Rate with single-period compounding | Standard for money markets |
| Forward rate identity | $1 + \tau F = P(T_1)/P(T_2)$ | Links discount factors to forwards |
| Par condition | PV = notional at inception | Enables bootstrapping |
| Banker's discount yield | $(F - P)/F \times 360/d$ | Bill quoting convention (not IRR) |
| FRA | Forward-starting deposit, net-settled | Direct constraint on forward curve |
| In advance | Rate known at period start (LIBOR) | Predictable cash flows |
| In arrears | Rate known at period end (SOFR) | Operational complexity |
| Fed probability | $(r_{expected} - r_0)/(r_1 - r_0)$ | Extract Fed action odds |
| Turn (special-date stub forward) | Implied overnight rate over a special-date stub (e.g., Dec 31) | Front-end funding can dislocate; must be modeled/hedged explicitly |

---

## Notation

| Symbol | Meaning | Units / Convention |
|--------|---------|-------------------|
| $P(t,T)$ | Discount factor (time-$t$ PV of 1 paid at $T$) | unitless; $P(T,T)=1$ |
| $\tau(T_1,T_2)$ | Year fraction for $[T_1,T_2]$ | years; depends on day count (often ACT/360 in USD money markets) |
| $L(t;T_1,T_2)$ | Simply-compounded forward rate for $[T_1,T_2]$ | per year; quote basis must be stated |
| $F(t;T_1,T_2)$ | Forward rate (same object as $L$ in this chapter) | per year |
| $K$ | FRA fixed rate | per year; same basis as $L$ |
| $N$ | Notional principal | currency |
| $q_{\text{disc}}$ | T-bill quoted discount rate (bank discount quote) | percent per year; ACT/360; applied to face; $Y = 100 - q_{\text{disc}}d/360$ |
| $Y$ | Cash price per \$100 face (T-bill) | dollars per \$100 face |
| $d$ | Days to maturity (bill) | calendar days |
| $r_i$ | Daily overnight rate on day $i$ | per year |
| $d_i$ | Days that rate $r_i$ applies | calendar days |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | Define the discount factor $P(0,T)$. | The time-0 price of receiving 1 unit at time $T$. |
| 2 | Define the year fraction $\tau(T,S)$. | The day-count year fraction between dates $T$ and $S$ (units: years). |
| 3 | State the deposit-implied discount factor formula. | $P(0,T) = 1/(1 + r \cdot \tau(0,T))$ (simple interest). |
| 4 | State the simple-forward identity. | $1 + \tau(T_1,T_2)\,f(t;T_1,T_2) = P(t,T_1)/P(t,T_2)$. |
| 5 | Express the forward rate $f$ explicitly. | $f(t;T_1,T_2) = (P(t,T_1)/P(t,T_2) - 1)/\tau(T_1,T_2)$. |
| 6 | What does "par" mean for a deposit? | Discounted maturity payoff equals notional invested. |
| 7 | What does "par" mean for an FRA? | The FRA has zero PV when $K$ equals the forward rate. |
| 8 | Write the FRA value in discount factors. | $V(t)=P(t,T_1)-P(t,T_2)-K\,\tau(T_1,T_2)\,P(t,T_2)=\tau(T_1,T_2)\,P(t,T_2)\,(f(t;T_1,T_2)-K)$. |
| 9 | Write the FRA settlement payoff. | $V(T_1)=\tau(T_1,T_2)\,(L_{T_1}-K)/(1+\tau(T_1,T_2)\,L_{T_1})$, where $L_{T_1}=L(T_1;T_1,T_2)$. |
| 10 | Define the T-bill bank discount quote $q_{\text{disc}}$. | $q_{\text{disc}} = \frac{360}{d}(100-Y)$ where $Y$ is cash price per \$100 face (units: percent per year; ACT/360). |
| 11 | Why isn't banker's discount yield an IRR? | It uses face (not price) in the denominator. |
| 12 | What day count does U.S. money market use? | Actual/360. |
| 13 | Describe the bootstrap logic. | Set instrument PV to par; solve sequentially for discount factors. |
| 14 | Why is interpolation needed? | Only finitely many instruments exist; DFs at all maturities aren't observable. |
| 15 | Formula to solve $P(0,T_2)$ from a par FRA. | $P(0,T_2) = P(0,T_1)/(1 + K \cdot \tau(T_1,T_2))$ |
| 16 | State a sign sanity check for deposits. | Higher rates → lower discount factors. |
| 17 | State a bound check for discount factors. | $P(0,T) > 0$ and typically $\leq 1$ for positive rates. |
| 18 | What does a STIR futures quote represent? | $Q = 100 \times (1 - \text{futures rate})$ |
| 19 | Why do futures rates differ from forward rates? | Daily settlement creates funding cost correlation; convexity adjustment needed. |
| 20 | What changed post-2007 in curve practice? | Single curve → multiple curves (separate discounting and forwarding). |
| 21 | What is "rate fixing in advance"? | Rate known at the start of the accrual period (LIBOR convention). |
| 22 | What is "rate fixing in arrears"? | Rate determined by compounding daily observations over the period; known only at period end (SOFR convention). |
| 23 | How do you extract implied Fed rate change probability from Fed Funds futures? | Calculate expected post-meeting rate from futures price, then $p = (r_{expected} - r_{no change})/(r_{change} - r_{no change})$. |
| 24 | Why doesn't SOFR capture corporate credit spread risk? | SOFR is a secured overnight rate (Treasury repo) with essentially no credit spread; corporate borrowing costs include credit premiums that SOFR doesn't hedge. |
| 25 | What is the year-end "turn"? | A special-date stub around year-end that can create a local hump/dip in very short-dated forwards. |
| 26 | What is the book DV01 convention? | $DV01 := PV(\text{rates down }1\text{bp})-PV(\text{base})$ for a stated bump object (units: currency per 1bp). |

---

## Mini Problem Set

1. Compute $P(0,1M)$ from a 1M deposit quote of 4.80% using ACT/360 and 31 days.

2. Using $P(0,1M) = 0.9960$ and $P(0,3M) = 0.9880$, compute $F(0;1M,3M)$ with $\tau = 62/360$.

3. A 90-day bill has banker's discount yield 6.00% and face 100. Compute price and $P(0,T)$.

4. Given $P(0,6M) = 0.9750$ and a par FRA 6x9 quote of 5.10% with $\tau = 90/360$, compute $P(0,9M)$.

5. Given $P(0,3M) = 0.9900$ and $P(0,6M) = 0.9750$, compute the par 3x6 FRA rate.

6. For $N = 50{,}000{,}000$, $\tau = 0.25$, $P(0,6M) = 0.9750$, and $F = 5.00\%$, compute FRA PV when $K = 4.85\%$.

7. Build a discount factor table at 1M, 3M, 6M, 12M from three deposit quotes and one FRA quote.

8. Bump the 3M deposit by +1 bp; recompute $P(0,3M)$ and the 3M–6M forward.

9. A bill is quoted with discount rate 8.00% for 91 days. What is the cash price per $100 face?

10. A STIR futures price is 95.25. Compute the implied futures rate.

11. Explain qualitatively why daily settlement creates a convexity adjustment.

12. Describe how curve construction changes from single-curve to multi-curve.

13. **Fed probability extraction:** January Fed Funds futures price is 94.51 (implying 5.49% average). The FOMC meets January 29 (assume January has 31 days). Current target is 5.50%. What is the implied probability of a 25bp cut? Assume only two outcomes: no change or 25bp cut.

14. **SOFR compounding:** Given three daily SOFR fixings: Day 1 = 5.00% (1 day), Day 2 = 5.02% (1 day), Day 3 = 5.01% (3 days, weekend). Compute the 5-day compounded rate.

15. **Year-end turn:** An OIS curve shows $P(0, \text{Dec 30}) = 0.99750$ and $P(0, \text{Jan 3}) = 0.99690$. The period spans 4 calendar days (including 2 weekend days). What is the implied overnight rate for this period? If the "normal" overnight rate is 5.00%, what is the turn premium?

---

### Solution Sketches (Questions 1–8)

1. $\tau = 31/360 = 0.08611$. $P = 1/(1 + 0.048 \times 0.08611) = 0.99588$.

2. $F = (0.9960/0.9880 - 1)/(62/360) = 0.00810/0.1722 = 4.70\%$.

3. Price $= 100(1 - 0.06 \times 90/360) = 100(1 - 0.015) = 98.50$. $P = 0.9850$.

4. $P(0,9M) = 0.9750/(1 + 0.051 \times 0.25) = 0.9750/1.01275 = 0.9627$.

5. $F = (0.9900/0.9750 - 1)/0.25 = 0.01538/0.25 = 6.15\%$.

6. $V = 50M \times 0.25 \times 0.9750 \times (0.0500 - 0.0485) = 50M \times 0.25 \times 0.9750 \times 0.0015 = \$18,281$.

7. Follow Example 6 methodology with given quotes.

8. If $r_{3M}$ increases by 1bp, $P(0,3M)$ decreases slightly. Forward from 3M to 6M increases (3M discount factor smaller → ratio larger → forward larger).

### Solution Sketches (Questions 13–15)

13. Implied average = 5.49%. Days before meeting (1-28): 28 days at 5.50%. Days after (29-31): 3 days at unknown rate.
$5.49\% = (28/31) \times 5.50\% + (3/31) \times r_{post}$
$r_{post} = (31 \times 5.49\% - 28 \times 5.50\%)/3 = (170.19\% - 154\%)/3 = 5.3967\%$
$p_{cut} = (5.50\% - 5.3967\%)/(5.50\% - 5.25\%) = 0.1033\%/0.25\% = 41.3\%$

14. Day 1: $1 + 0.0500/360 = 1.000138889$; Day 2: $1 + 0.0502/360 = 1.000139444$; Day 3: $1 + 0.0501 \times 3/360 = 1.000417500$.
Product: $1.000695903$. Rate = $(0.000695903) \times 360/5 = 5.01\%$.

15. Forward factor: $0.99750/0.99690 = 1.000602$. Implied rate: $(1.000602 - 1) \times 360/4 = 5.42\%$. Turn premium: $5.42\% - 5.00\% = 42$ bp.

---

## References

- (Bruce Tuckman, *Fixed Income Securities*, “Fed Funds”; “Fed Funds Futures”)
- (John C. Hull, *Options, Futures, and Other Derivatives*, “Day Counts”; “Price Quotations of U.S. Treasury Bills”; “Convexity Adjustments”)
- (Marek Musiela and Marek Rutkowski, *Martingale Methods in Financial Modelling*, “Treasury Bill Futures”)
- (Leif B. G. Andersen and Vladimir V. Piterbarg, *Interest Rate Modeling*, “Yield Curve Construction and Risk Management”)
- (Robert A. Jarrow, *Modeling Fixed Income Securities and Interest Rate Options* (3rd ed.), “15.1 Simple Interest Rates”)
- (Ali Hirsa, *Computational Methods in Finance* (2nd ed.), “Forward Rate Agreement (FRA)”)
- (John C. Hull, *Risk Management and Financial Institutions*, “The OIS Rate”)
