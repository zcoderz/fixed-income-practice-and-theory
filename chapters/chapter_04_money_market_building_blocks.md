# Chapter 4: Money-Market Building Blocks (The Shortest Curve Points)

---

## Introduction

Every yield curve begins somewhere. Before you can bootstrap to 30-year swap rates or price a 10-year Treasury bond, you need the first few discount factors—the anchors that pin down the present value of the nearest cashflows. These anchors come from the **money market**: deposits, Treasury bills, forward rate agreements (FRAs), and short-term interest rate futures.

The money-market segment of the curve looks deceptively simple. Instruments mature in weeks or months, not years. The math involves little more than present value with simple interest. Yet this is precisely where **day-count conventions matter most**: a one-day difference in accrual fraction can be as significant as a basis point difference in rate. At the short end, the plumbing cannot be approximate—it must be exact.

This chapter covers:

1. **Core concepts**: Discount factors, accrual factors, simple rates, and the forward-rate identity that ties them together
2. **Instrument mechanics**: What deposits, bills, and FRAs actually quote and how each pins down a curve point
3. **Mathematical derivations**: Step-by-step with unit checks and sanity tests
4. **Bootstrapping**: How to build the short end from market quotes
5. **Worked examples**: Eight detailed numeric calculations

We work primarily in a **single-curve setting** for pedagogy. The multi-curve reality of post-crisis practice—separate discounting and forwarding curves—is previewed but deferred to later chapters.

> **Connection to prior chapters:** Chapter 1 established day counts and compounding as the "unit system" of interest rates. Chapter 2 defined discount factors as the primitive pricing object. Chapter 3 introduced the zero-forward-par triangle. This chapter shows how actual market instruments—deposits, bills, FRAs—generate those first discount factors and forward rates.

---

## 4.1 Core Concepts

### 4.1.0 The Money Market Ecosystem

Before diving into the math, it helps to understand the "Credit Hierarchy" of the short end. Money markets are not a monolith; they are strata of trust.

> **Visualizing the Credit Pyramid**
>
> *   **Tier 1 (Risk-Free)**: **Fed Funds / IOER**. The central bank's own balance sheet. The ultimate safe haven.
> *   **Tier 2 (Sovereign)**: **T-Bills**. Backed by the Treasury. Effectively risk-free, but distinct from the Fed.
> *   **Tier 3 (Secured)**: **Repo**. Buying a Treasury and agreeing to sell it back. Collateralized borrowing. Safe because of the collateral.
> *   **Tier 4 (Unsecured Banks)**: **Eurodollar / Term LIBOR / CP**. "I promise to pay you back." Trust-based.
> *   **Tier 5 (Corporate)**: **Commercial Paper**. Unsecured corporate debt.
>
> **The Lesson of 2008:** In normal times, these rates move together. In a crisis, trust evaporates. The "Pyramid" pulled apart: Tier 4/5 rates spiked (Libor soared) while Tier 1/2 rates plummeted (Flight to Quality).

### 4.1.1 The Discount Factor $P(0,T)$

The discount factor $P(0,T)$ is the time-0 value of receiving one unit of currency at time $T$. It is the fundamental building block of fixed-income pricing.

**Formal definition:** $P(0,T)$ is the price today of a zero-coupon bond that pays 1 at maturity $T$. As Tuckman puts it, "The discount factor for a particular term gives the value today, or the present value of one unit of currency to be received at the end of that term."

**Intuition:** Think of $P(0,T)$ as the "present-value weight" applied to any cashflow at time $T$. At the short end of the curve, discount factors are close to 1—a payment due in 30 days is worth almost its face value—but even small deviations matter because short-dated instruments have tight bid-ask spreads and high trading frequency.

**Practice:** Front-end discount factors are the first "curve nodes" that anchor PVs of very short cashflows and the earliest forward rates used in floating-rate products.

### 4.1.2 The Accrual Factor $\alpha(T,S)$ and Day Count

Hull provides a clean definition: "The day count defines the way in which interest accrues over time." The accrual factor $\alpha(T,S)$ converts an annualized rate into an interest amount over the period $[T,S]$:

$$\text{Interest} = \text{Rate} \times \alpha(T,S) \times \text{Notional}$$

**Why this matters at the short end:** The front end is "day-count dominated." When instruments mature in weeks rather than years, a one-day error in the accrual fraction can matter as much as a basis point error in the rate. As Hull emphasizes, "the interest earned in a whole year of 365 days is $365/360$ times the quoted rate" under Actual/360—a seemingly small detail that compounds into real dollars.

**Convention variability:** Day-count conventions differ by country and instrument. Hull notes: "Conventions vary from country to country and from instrument to instrument. For example, money market instruments are quoted on an actual/365 basis in Australia, Canada, and New Zealand. LIBOR is quoted on an actual/360 for all currencies except sterling, for which it is quoted on an actual/365 basis."

In the U.S., money-market instruments (including SOFR and Fed Funds) commonly use **Actual/360**: interest accrued over $d$ days is $(d/360) \times \text{rate}$.

### 4.1.3 Simple (Money-Market) Rates

Money-market instruments quote **simple rates** over their term—compounding frequency equals the instrument's maturity. A deposit of size 1 over $[T, T+\tau]$ returns $1 + \tau L(T; T, T+\tau)$ at maturity. Tuckman explains that "lending \$1 for $d$ days at a rate of $r$ will earn the lender an interest payment of $rd/360$ dollars at the end of the $d$ days" under the actual/360 convention.

**From rate to discount factor:** If you know a deposit rate $r$ and the corresponding accrual fraction $\alpha$, you can directly compute the implied discount factor:

$$\boxed{P(0,T) = \frac{1}{1 + r \cdot \alpha(0,T)}}$$

This formula captures the essence of short-end curve construction: a deposit quote is "directly a discount factor" once you apply the correct day-count convention.

### 4.1.4 The Forward Rate Identity

For any future period $[T,S]$, the simply-compounded forward rate $L(t;T,S)$ satisfies:

$$\boxed{1 + \alpha(T,S) \cdot L(t;T,S) = \frac{P(t,T)}{P(t,S)}}$$

**Intuition:** The forward rate is the breakeven simple interest rate over $[T,S]$ implied by the discount curve. If you can borrow at $P(t,T)$ and lend at $P(t,S)$, the forward rate is the return you lock in.

Andersen defines this identity as the foundation for all forward-rate instruments: "the simply-compounded forward rate satisfies $1 + \tau L = P(t,T)/P(t,S)$."

**Solving for the forward rate:**

$$\boxed{F(0;T_1,T_2) = \frac{\frac{P(0,T_1)}{P(0,T_2)} - 1}{\alpha(T_1,T_2)}}$$

### 4.1.5 What "Par" Means and How It Pins Down a Curve Point

An instrument is **at par** if its present value equals zero (for a derivative) or equals the notional invested (for a deposit or bill purchase at its quoted price).

For a deposit, "par" means the discounted maturity payoff equals the amount invested:

$$1 = P(0,T) \cdot (1 + r \cdot \alpha(0,T))$$

This equation is the key to bootstrapping: each par quote lets you solve for one unknown discount factor.

---

## 4.2 Instruments at the Short End: What Is Quoted and What It Pins Down

### 4.2.1 Deposits and Money-Market Instruments

#### The Simple Interest Deposit Quote

A money-market deposit quotes a simple annualized rate $r$ for a maturity $T$, together with a day-count convention that determines $\alpha(0,T)$.

**What the quote pins down:** The discount factor $P(0,T)$ via the par condition:

$$1 = P(0,T) \cdot (1 + r \cdot \alpha(0,T))$$

Solving:

$$\boxed{P(0,T) = \frac{1}{1 + r \cdot \alpha(0,T)}}$$

Hull explains that "the actual/360 day count is used for money market instruments in the United States. This indicates that the reference period is 360 days. The interest earned during part of a year is calculated by dividing the actual number of elapsed days by 360 and multiplying by the rate."

#### Accrual and Settlement Conventions

Several operational conventions affect how deposit quotes translate to discount factors:

1. **Day count:** U.S. money markets use Actual/360; other markets may use Actual/365.
2. **Spot lag:** Many interbank rates have a settlement lag (e.g., T+2 for legacy USD LIBOR, T+0 for Fed Funds, T+1 for some Bills).
3. **Business day adjustments:** Modified following is common for date roll conventions.

For pedagogical examples in this chapter, we ignore spot lags and assume year fractions are given directly.

### 4.2.2 Treasury Bills: Discount Instruments

> **Repo 101: The Plumbing of Wall Street**
>
> A **Repurchase Agreement (Repo)** is legally a *sale* and a *repurchase*, but economically a **collateralized loan**.
> *   **Leg 1**: I sell you a Treasury bond for $100 today.
> *   **Leg 2**: I agree to buy it back tomorrow for $100.01.
> *   **Economics**: I borrowed $100 from you overnight at a 1bp interest rate, using the bond as collateral.
>
> Repo rates (like SOFR) form the bedrock of the modern "Tier 3" secured funding market.

#### Bill Price Quoting vs. Discount Yield Quoting

Hull explains: "The prices of money market instruments are sometimes quoted using a discount rate. This is the interest earned as a percentage of the final face value rather than as a percentage of the initial price paid for the instrument."

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
> **Rule**: Investment Yield is **always** higher than Discount Yield because Price < Face. The gap explodes as rates rise.

A U.S. Treasury bill with 91 days to maturity quoted at "8" means the discount rate is 8% per 360 days, calculated on the face value (not the price paid). Hull provides a concrete example: "If the price of a 91-day Treasury bill is quoted as 8, this means that the rate of interest earned is 8% of the face value per 360 days. Suppose that the face value is \$100. Interest of \$2.0222 (= \$100 × 0.08 × 91/360) is earned over the 91-day life."

**Banker's discount yield** $y_{BD}$ is defined as:

$$y_{BD} = \left(\frac{\text{Face} - \text{Price}}{\text{Face}}\right) \times \frac{360}{d}$$

where $d$ is the number of days to maturity.

**Converting to price:** Solving for the cash price:

$$\boxed{\text{Price} = \text{Face} \times \left(1 - y_{BD} \times \frac{d}{360}\right)}$$

Hull provides the general relationship between cash price $Y$ and quoted discount rate $P$ (in his notation):

$$P = \frac{360}{n}(100 - Y)$$

where $n$ is the remaining life in calendar days. Note that Hull uses $P$ for the quoted discount rate and $Y$ for the cash price—the opposite convention from some other texts.

#### From Bill Price to Discount Factor

Once you have the cash price for face value $F$:

$$P(0,T) = \frac{\text{Price}}{F}$$

#### Caution: Multiple Yield Conventions Exist

The **ask yield** (often called **Bond Equivalent Yield** or **BEY**) differs from the banker's discount yield. Hull explains that the discount yield "corresponds to a true rate of interest of 2.0222/(100 - 2.0222) = 2.064% for the 91-day period"—higher than the 2.0222% discount on face because the denominator is the price paid, not the face value.

$$y_{\text{ask}} = \left(\frac{\text{Face} - \text{Price}}{\text{Price}}\right) \times \frac{365}{d}$$

Note the differences:
- Denominator: price (not face)
- Annualization: 365 (not 360)

The banker's discount yield is a convenient quoting convention, **not** an internal rate of return.

### 4.2.3 Forward Rate Agreements (FRAs)

#### FRA as a Forward-Starting Deposit

Hull defines an FRA as "an agreement to exchange a predetermined fixed rate for a reference rate that will be observed in the market at a future time. Both rates are applied to a specified principal, but the principal itself is not exchanged."

An FRA references the future floating rate for period $[T, T+\tau]$ and exchanges (in net form) the difference between floating and a fixed contract rate $K$.

> **Analogy: "Pre-Ordering" Your Loan**
>
> An FRA is like ordering a pizza for next month at today's fixed price.
> *   **Lock-In**: You agree to pay 5% for a 3-month loan starting in June.
> *   **Scenario A (Rates Rise to 7%)**: The market loan costs 7%. You are happy! The FRA seller pays you the difference (2%). Net cost = 5%.
> *   **Scenario B (Rates Fall to 3%)**: The market loan costs 3%. You are sad. You must pay the FRA seller the difference (2%). Net cost = 5%.
>
> **Result**: No matter where rates go, your net effective rate is locked at 5%.

#### FRA Value in Terms of Discount Factors

The value of an FRA at time $t \leq T$ for unit notional can be written:

$$V_{\text{FRA}}(t) = P(t,T) - P(t,T+\tau) - \tau K \cdot P(t,T+\tau)$$

This can be rewritten as:

$$V_{\text{FRA}}(t) = \tau \cdot P(t,T+\tau) \cdot \left(L(t;T,T+\tau) - K\right)$$

where $L(t;T,T+\tau)$ is the forward rate from the discount curve.

Hull notes that "an FRA can be valued by assuming that the forward interest rate for the underlying reference rate will be the one that determines the exchange."

#### The Par FRA Rate Equals the Forward Rate

Setting $V_{\text{FRA}}(t) = 0$ and solving for the fixed rate $K$:

$$\boxed{K^* = L(t;T,T+\tau) = \frac{1}{\tau}\left(\frac{P(t,T)}{P(t,T+\tau)} - 1\right)}$$

The par FRA rate equals the implied forward rate—a fundamental result that makes FRAs direct constraints on the forward curve. Hull states: "When the fixed rate equals the relevant forward rate the value of an FRA is zero."

#### FRA Settlement Payoff

At settlement (time $T$), the net payment is:

$$\boxed{V_{\text{FRA}}(T) = \frac{\tau \cdot (L(T;T,T+\tau) - K)}{1 + \tau \cdot L(T;T,T+\tau)}}$$

The denominator discounts the payment from $T+\tau$ back to $T$. Hull notes in a footnote: "In practice, because LIBOR is determined in advance of a period, the payment would be made at time $T$ and equal to the present value of the cash flow discounted for the period at the observed rate."

### 4.2.4 STIR Futures: Forward-Rate Instruments (Preview Only)

#### Quote Convention

Andersen describes Eurodollar (and similarly SOFR) futures as contracts where "at time 0, a Eurodollar futures contract can be entered into at no upfront cost, but with an implicit obligation of the holder to pay at time $T$ per unit of notional $1 - F(0,T,T+\tau)$ in return for the payout $1 - L(T,T,T+\tau)$."

The quoted price is typically:

$$Q(t;T) = 100 \times (1 - \text{futures rate})$$

For example, a quote of 94.67 implies a futures rate of 5.33%.

#### Futures vs. Forwards: The Convexity Adjustment

Because futures are marked to market daily while FRAs settle at maturity, the futures-implied rate generally differs from the corresponding forward rate. Andersen explains the intuition directly:

> "Due to the adverse behavior of funding costs and reinvestment gains, we would expect the purchaser of a Eurodollar futures contract to pay less for these instruments than for a comparable instrument without daily mark-to-market. Consequently, we would expect the futures rate to be above the corresponding forward rate."

The mechanism is as follows: under rising interest rates, the futures holder must make margin payments at precisely the moment when borrowing costs are highest. Conversely, when rates fall, received margin is reinvested at lower rates. This systematic disadvantage means futures rates must exceed forward rates to compensate.

Hull similarly notes that daily settlement creates correlation between rate moves and funding costs that requires a **convexity adjustment** to convert futures rates to forward rates.

This chapter does not derive the convexity adjustment—it is deferred to Part V on futures.

---

## 4.3 Mathematical Derivations

### 4.3.1 Deposit Quote → Discount Factor

**Setup:**
- Simple interest over $[0,T]$
- Unit principal invested at $t = 0$
- No default; single-curve framework

**Cashflow:** The deposit payoff at $T$ is $1 + r \cdot \alpha(0,T)$.

**Par condition:** The present value at inception equals 1:

$$1 = P(0,T) \cdot (1 + r \cdot \alpha(0,T))$$

**Solve for the discount factor:**

$$\boxed{P(0,T) = \frac{1}{1 + r \cdot \alpha(0,T)}}$$

**Unit check:** Rate $r$ has units "per year"; $\alpha$ has units "years"; so $r \cdot \alpha$ is dimensionless, and $P$ is dimensionless. ✓

**Sanity checks:**
- If $r$ increases, the denominator increases, so $P(0,T)$ decreases. ✓
- As $T \to 0$, $\alpha(0,T) \to 0$, so $P(0,T) \to 1$. ✓

### 4.3.2 Forward Rate from Discount Factors

From the forward-rate identity:

$$1 + \alpha(T_1,T_2) \cdot F(0;T_1,T_2) = \frac{P(0,T_1)}{P(0,T_2)}$$

Solving for the forward rate:

$$\boxed{F(0;T_1,T_2) = \frac{\frac{P(0,T_1)}{P(0,T_2)} - 1}{\alpha(T_1,T_2)}}$$

**Unit check:** Numerator is dimensionless; dividing by $\alpha$ (years) gives "per year." ✓

**Sanity check:** If $P(0,T_2)$ is smaller (more discounting to $T_2$), the forward rate is larger. ✓

### 4.3.3 FRA Par Rate Equals the Forward Rate

From the FRA value expression:

$$V_{\text{FRA}}(t) = P(t,T) - P(t,T+\tau) - \tau K \cdot P(t,T+\tau)$$

Setting $V_{\text{FRA}}(t) = 0$:

$$P(t,T) = P(t,T+\tau) \cdot (1 + \tau K)$$

Solving for $K$:

$$K = \frac{1}{\tau}\left(\frac{P(t,T)}{P(t,T+\tau)} - 1\right)$$

But by the forward-rate identity:

$$L(t;T,T+\tau) = \frac{1}{\tau}\left(\frac{P(t,T)}{P(t,T+\tau)} - 1\right)$$

Therefore:

$$\boxed{K^* = L(t;T,T+\tau) = F(t;T,T+\tau)}$$

### 4.3.4 FRA Time-0 Present Value

For notional $N$ and fixed rate $K$:

$$\boxed{V_{\text{FRA}}(0) = N \cdot \tau \cdot P(0,T+\tau) \cdot (F - K)}$$

where $F = L(0;T,T+\tau)$ is the forward rate.

**Sanity checks:**
- If $K = K^*$ (par), then $V_{\text{FRA}}(0) = 0$. ✓
- For the fixed-rate payer, value increases when the forward rate increases. ✓

---

## 4.4 Bootstrapping the Short End

### 4.4.1 The Sequential Logic

Curve construction is an inverse problem: given prices of traded instruments, recover the discount factors. As Andersen notes, "only a finite set of traded instruments is observable; discount bonds for all maturities are not directly observable"—hence the need for both bootstrapping and interpolation.

**The short-end bootstrap:**

**Step 1: Initialize.** $P(0,0) = 1$.

**Step 2: Use deposits (or bills) for the earliest nodes.** For each maturity $T$ with deposit quote $r(0,T)$:

$$P(0,T) = \frac{1}{1 + r(0,T) \cdot \alpha(0,T)}$$

For bills quoted via discount yield, first convert quote → price, then $P(0,T) = \text{price}/\text{face}$.

**Step 3: Use FRAs to extend further.** If $P(0,T_1)$ is known and you have a par FRA quote $K$ for $[T_1,T_2]$, then:

$$\boxed{P(0,T_2) = \frac{P(0,T_1)}{1 + K \cdot \alpha(T_1,T_2)}}$$

This follows directly from the forward-rate identity.

**Step 4: Interpolate between nodes.** Between instrument maturities, choose an interpolation scheme (on zero rates, log discount factors, or forward rates). This is a modeling choice with implications for smoothness and locality of perturbations.

### 4.4.2 Single-Curve vs. Multi-Curve (Preview)

Andersen's curve construction discussion notes that pre-2007 practice commonly used a single LIBOR curve; after the crisis, a single curve was no longer adequate. As Andersen explains: "post-crisis a nontrivial basis between index and discounting curves has emerged in the US. For simplicity of exposition we proceed in this section with [the single-curve assumption], but the index-discounting basis in the US could be easily incorporated into the algorithm."

In modern practice, "the Fed funds rate, the overnight rate used for balances of bank deposits with the Federal reserve, is often considered the closest proxy to the risk-free rate in the US." OIS curves provide discounting, while separate projection curves estimate future LIBOR/SOFR fixings.

This chapter remains single-curve to explain the mechanics. Multi-curve bootstrapping—where OIS provides the discounting curve and term rates provide projection curves—is covered in Part IV.

---

## 4.5 Rate Sensitivity at the Short End

### 4.5.1 Sensitivity of Discount Factor to Rate

For a deposit-implied discount factor:

$$P(0,T) = \frac{1}{1 + r \cdot \alpha}$$

The sensitivity to the quoted rate is:

$$\frac{\partial P}{\partial r} = -\frac{\alpha}{(1 + r \cdot \alpha)^2}$$

**Interpretation:** The short-end discount factor is most sensitive to (i) larger accrual fractions and (ii) lower rate levels. Even small $\alpha$ matters because front-end PVs are dominated by short-dated cashflows.

### 4.5.2 Locality of Short-End Perturbations

A key property of sequential bootstrapping: changing a short-dated quote primarily affects the nearest nodes and the forwards that depend on them. Andersen describes "bucket exposure" analysis where bumping a specific forward-rate segment reveals the locality of curve shocks.

This locality intuition underlies key-rate DV01 analysis, covered in Chapter 14.

---

## 4.6 Worked Examples

### Numerical Conventions (Examples 1–8)

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

| Tenor | Days | $\alpha$ |
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
$$F(0;T_1,T_2) = \frac{\frac{P(0,T_1)}{P(0,T_2)} - 1}{\alpha(T_1,T_2)}$$

With:
- $P(0,1M) = 0.99585062$
- $P(0,3M) = 0.98716683$
- $\alpha(1M,3M) = 60/360 = 0.16666667$

**Step 1:** Compute the ratio:
$$\frac{P(0,1M)}{P(0,3M)} = \frac{0.99585062}{0.98716683} = 1.00879668$$

**Step 2:** Compute the forward:
$$F(0;1M,3M) = \frac{1.00879668 - 1}{0.16666667} = \frac{0.00879668}{0.16666667} = 0.05278008$$

$$\boxed{F(0;1M,3M) \approx 5.2780\%}$$

---

### Example 3: Bill Discount Yield → Price/DF

**Given:**
- 91-day Treasury bill
- Face value $F = 100$
- Banker's discount yield $y_{BD} = 5.20\%$

**Step 1:** Convert to price:
$$\text{Price} = 100 \times \left(1 - 0.0520 \times \frac{91}{360}\right)$$

Compute:
- $91/360 = 0.25277778$
- $0.0520 \times 0.25277778 = 0.01314444$

$$\text{Price} = 100 \times (1 - 0.01314444) = 98.6855556$$

**Step 2:** Implied discount factor:
$$P(0,T) = \frac{98.6855556}{100} = 0.986855556$$

**Convention note:** This discount factor reflects the banker's discount quote. The true yield-to-maturity (or BEY) would be computed from price to face as:
$$\text{True return (BEY)} = \frac{100 - 98.686}{98.686} \times \frac{365}{91} = 5.34\%$$

Note that the BEY is higher than the discount yield because (1) the denominator is the lower price paid rather than face, and (2) it uses 365-day annualization.

---

### Example 4: FRA Par Rate from Discount Factors

**Add a 6M deposit:** $r_{6M} = 5.30\%$, $\alpha(0,6M) = 0.5$

**Step 1:** Compute $P(0,6M)$:
$$P(0,6M) = \frac{1}{1 + 0.0530 \times 0.5} = \frac{1}{1.0265} = 0.97418400$$

**Step 2:** Compute the 3M–6M par FRA rate:
- $\alpha(3M,6M) = 90/360 = 0.25$
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
| Tenor | Type | Quote | $\alpha$ |
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

$$P(0,12M) = \frac{P(0,6M)}{1 + K \cdot \alpha(6M,12M)} = \frac{0.97418400}{1 + 0.054 \times 0.5} = \frac{0.97418400}{1.027} = 0.94857173$$

**Discount Factor Table:**

| Maturity | Days | $\alpha(0,T)$ | Source | Quote | $P(0,T)$ |
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

**Observation:** Futures rate (5.33%) is slightly above the forward rate (5.33%)—consistent with the convexity adjustment intuition that futures rates exceed forward rates due to daily settlement effects.

---

## 4.7 Practical Notes

### 4.7.1 Quoting Conventions to Watch

**Simple vs. compounded:** Money-market rates are simple; mixing them with continuously compounded rates without conversion causes errors.

**Day count matters:** ACT/360 is standard for U.S. money markets, but ACT/365 applies in some jurisdictions. Hull emphasizes these vary "from country to country and from instrument to instrument."

**Bill conventions are not interchangeable:** Banker's discount yield uses face in the denominator and 360-day annualization. The ask yield (BEY) uses price and 365-day annualization. Neither is an IRR.

**Basis points vs. decimals:** $1\text{ bp} = 0.0001$ in decimal. Many front-end pricing errors are unit mistakes.

### 4.7.2 Common Implementation Pitfalls

| Pitfall | Consequence |
|---------|-------------|
| Wrong day count | Systematic accrual error |
| Mixing ACT/360 and ACT/365 | Inconsistent discount factors |
| Ignoring spot lags | Wrong settlement dates |
| Bootstrap ordering errors | Solving for wrong unknowns |
| Unit confusion (bp vs %) | Order-of-magnitude errors |

### 4.7.3 Verification Tests

**Repricing test for deposits:** Verify $P(0,T) \cdot (1 + r \cdot \alpha) = 1$.

**Repricing test for FRAs:** Verify par FRA has PV ≈ 0.

**Sign sanity:** Higher rates → lower discount factors.

**Bound checks:** $P(0,T) > 0$, typically $P(0,T) \leq 1$ for non-negative rates.

---

## Summary

1. **Short-end curve construction** begins with instruments whose PV-to-par conditions map directly to discount factors.

2. **A simple deposit** quoted at rate $r$ with accrual $\alpha$ implies $P(0,T) = 1/(1 + r\alpha)$.

3. **Day count is essential:** Hull notes that "the actual/360 day count is used for money market instruments in the United States."

4. **The forward-rate identity** $1 + \alpha F = P(0,T_1)/P(0,T_2)$ ties discount factors to forward rates.

5. **An FRA is a forward-starting deposit** in net-settlement form.

6. **The par FRA rate equals the forward rate** implied by the discount curve.

7. **Bill discount yields** are quoting conventions, not return measures. Convert to price first, then to discount factor.

8. **STIR futures quotes** map to implied rates; daily settlement creates a convexity adjustment (preview).

9. **Bootstrapping** is sequential: use each instrument to solve for one new unknown.

10. **Post-crisis practice** uses multiple curves; single-curve is pedagogical scaffolding.

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| Discount factor $P(0,T)$ | Price today of $1 at time $T$ | Fundamental pricing primitive |
| Accrual factor $\alpha$ | Year fraction under day-count convention | Converts rate to interest amount |
| Simple rate | Rate with single-period compounding | Standard for money markets |
| Forward rate identity | $1 + \alpha F = P(T_1)/P(T_2)$ | Links discount factors to forwards |
| Par condition | PV = notional at inception | Enables bootstrapping |
| Banker's discount yield | $(F - P)/F \times 360/d$ | Bill quoting convention (not IRR) |
| FRA | Forward-starting deposit, net-settled | Direct constraint on forward curve |

---

## Notation for This Chapter

| Symbol | Definition |
|--------|------------|
| $P(t,T)$ | Discount factor: time-$t$ value of 1 at $T$ |
| $\alpha(T,S)$ | Year fraction from $T$ to $S$ |
| $L(t;T,S)$ | Simply-compounded forward rate over $[T,S]$ |
| $F(0;T,S)$ | Same as $L(0;T,S)$ |
| $K$ | FRA fixed rate |
| $\tau$ | Accrual period length |
| $N$ | Notional principal |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | Define the discount factor $P(0,T)$. | The time-0 price of receiving 1 unit at time $T$. |
| 2 | Define the accrual factor $\alpha(T,S)$. | The year fraction between $T$ and $S$ under a day-count convention. |
| 3 | State the deposit-implied discount factor formula. | $P(0,T) = 1/(1 + r \cdot \alpha)$ |
| 4 | State the forward-rate identity. | $1 + \alpha \cdot F = P(0,T_1)/P(0,T_2)$ |
| 5 | Express the forward rate $F$ explicitly. | $F = (P(0,T_1)/P(0,T_2) - 1)/\alpha(T_1,T_2)$ |
| 6 | What does "par" mean for a deposit? | Discounted maturity payoff equals notional invested. |
| 7 | What does "par" mean for an FRA? | The FRA has zero PV when $K$ equals the forward rate. |
| 8 | Write the FRA value in discount factors. | $V(t) = P(t,T) - P(t,T+\tau) - \tau K \cdot P(t,T+\tau)$ |
| 9 | Write the FRA settlement payoff. | $V(T) = \tau(L - K)/(1 + \tau L)$ |
| 10 | Define banker's discount yield. | $((F - P)/F) \times (360/d)$ |
| 11 | Why isn't banker's discount yield an IRR? | It uses face (not price) in the denominator. |
| 12 | What day count does U.S. money market use? | Actual/360. |
| 13 | Describe the bootstrap logic. | Set instrument PV to par; solve sequentially for discount factors. |
| 14 | Why is interpolation needed? | Only finitely many instruments exist; DFs at all maturities aren't observable. |
| 15 | Formula to solve $P(0,T_2)$ from a par FRA. | $P(0,T_2) = P(0,T_1)/(1 + K \cdot \alpha)$ |
| 16 | State a sign sanity check for deposits. | Higher rates → lower discount factors. |
| 17 | State a bound check for discount factors. | $P(0,T) > 0$ and typically $\leq 1$ for positive rates. |
| 18 | What does a STIR futures quote represent? | $Q = 100 \times (1 - \text{futures rate})$ |
| 19 | Why do futures rates differ from forward rates? | Daily settlement creates funding cost correlation; convexity adjustment needed. |
| 20 | What changed post-2007 in curve practice? | Single curve → multiple curves (separate discounting and forwarding). |

---

## Mini Problem Set

1. Compute $P(0,1M)$ from a 1M deposit quote of 4.80% using ACT/360 and 31 days.

2. Using $P(0,1M) = 0.9960$ and $P(0,3M) = 0.9880$, compute $F(0;1M,3M)$ with $\alpha = 62/360$.

3. A 90-day bill has banker's discount yield 6.00% and face 100. Compute price and $P(0,T)$.

4. Given $P(0,6M) = 0.9750$ and a par FRA 6x9 quote of 5.10% with $\alpha = 90/360$, compute $P(0,9M)$.

5. Given $P(0,3M) = 0.9900$ and $P(0,6M) = 0.9750$, compute the par 3x6 FRA rate.

6. For $N = 50{,}000{,}000$, $\tau = 0.25$, $P(0,6M) = 0.9750$, and $F = 5.00\%$, compute FRA PV when $K = 4.85\%$.

7. Build a discount factor table at 1M, 3M, 6M, 12M from three deposit quotes and one FRA quote.

8. Bump the 3M deposit by +1 bp; recompute $P(0,3M)$ and the 3M–6M forward.

9. A bill is quoted with discount rate 8.00% for 91 days. What is the cash price per $100 face?

10. A STIR futures price is 95.25. Compute the implied futures rate.

11. Explain qualitatively why daily settlement creates a convexity adjustment.

12. Describe how curve construction changes from single-curve to multi-curve.

---

### Solution Sketches (Questions 1–6)

1. $\alpha = 31/360 = 0.08611$. $P = 1/(1 + 0.048 \times 0.08611) = 0.99588$.

2. $F = (0.9960/0.9880 - 1)/(62/360) = 0.00810/0.1722 = 4.70\%$.

3. Price $= 100(1 - 0.06 \times 90/360) = 100(1 - 0.015) = 98.50$. $P = 0.9850$.

4. $P(0,9M) = 0.9750/(1 + 0.051 \times 0.25) = 0.9750/1.01275 = 0.9627$.

5. $F = (0.9900/0.9750 - 1)/0.25 = 0.01538/0.25 = 6.15\%$.

6. $V = 50M \times 0.25 \times 0.9750 \times (0.0500 - 0.0485) = 50M \times 0.25 \times 0.9750 \times 0.0015 = \$18,281$.

---

## Source Map

### (A) Verified Facts (Source-Backed)

| Fact | Source |
|------|--------|
| Discount factor = zero-coupon bond price | Tuckman Ch 1, Andersen Vol 1 Ch 4 |
| Simple deposit relation $P = 1/(1+r\alpha)$ | Hull Ch 4, Andersen Vol 1, Tuckman Ch 4 |
| Forward rate identity $1 + \alpha L = P(T)/P(S)$ | Andersen Vol 1 Ch 4 (Eq. 4.2), Hull Ch 4 |
| FRA value expression in discount factors | Andersen Vol 1 Ch 5, Hull Ch 4 |
| Par FRA rate equals forward rate | Andersen Vol 1, Hull Ch 4 |
| ACT/360 for U.S. money markets | Hull Ch 6, Tuckman Ch 4 |
| Banker's discount yield formula for bills | Hull Ch 6 |
| Hull's bill formula $P = (360/n)(100 - Y)$ | Hull Ch 6 |
| Eurodollar/SOFR futures quote convention $100(1-\text{rate})$ | Andersen Vol 1 Ch 4 |
| Daily settlement creates convexity adjustment | Andersen Vol 1 Ch 4, Hull Ch 6 |
| Futures rate exceeds forward rate | Andersen Vol 1 Ch 4 |
| Post-crisis: multi-curve required | Andersen Vol 1 Ch 6 |
| Day count conventions vary by country | Hull Ch 6 |
| FRA settlement discounts payment to fixing date | Hull Ch 4 (footnote) |
| BEY uses price in denominator, 365-day annualization | Hull Ch 6 |

### (B) Reasoned Inference (Derived from A)

- Bootstrap sequence follows from par conditions: each par instrument pins down one unknown DF.
- Sensitivity $\partial P/\partial r = -\alpha/(1+r\alpha)^2$ is algebraic differentiation of the deposit formula.
- Locality of curve shocks follows from sequential bootstrap structure.
- Futures rate > forward rate follows from Andersen's funding cost argument.

### (C) Flagged Uncertainties

- **Exact settlement conventions:** FRA payment timing, spot lags, and business-day conventions vary by market and contract. Hull notes payment at $T$ with discounting; others may pay at $T+\tau$.
- **Day-count variations:** LIBOR conventions differ by currency (Hull Ch 6); specific jurisdictions may vary.
- **Convexity adjustment magnitude:** This chapter previews the concept but does not derive or quantify it.
- **Post-LIBOR conventions:** As markets transition to SOFR/SONIA, specific FRA and futures conventions continue to evolve.

---

*This chapter establishes the building blocks for short-end curve construction. The discount factors derived here—from deposits, bills, and FRAs—form the foundation for all subsequent pricing and risk analysis. Multi-curve extensions and convexity adjustments are covered in later chapters.*
