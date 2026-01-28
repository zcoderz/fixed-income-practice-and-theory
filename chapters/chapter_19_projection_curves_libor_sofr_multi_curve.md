# Chapter 19: Projection Curves — LIBOR, SOFR, and the Multi-Curve Framework

---

## Introduction

For decades, the fixed income market operated on a beautifully simple assumption: there was only one yield curve. Whether you were pricing a government bond, a corporate loan, or an interest rate swap, the fundamental "time value of money" was derived from a single set of inputs—typically the LIBOR interbank curve. This framework relied on the premise that major financial institutions lent to each other at rates that were effectively risk-free, and that a 3-month loan could be seamlessly replicated by rolling three 1-month loans.

That assumption died in August 2007.

As the Global Financial Crisis unfolded, the "law of one price" for money broke down. Banks became wary of lending to one another, causing the spread between secured overnight rates (like the Fed Funds rate) and unsecured term rates (like LIBOR) to explode. Andersen and Piterbarg document this shift precisely: "While the spread between the Fed funds rate and 3 month Libor rate used to be very small—in the order of a few basis points—after September 2007 it went up to as much as **275 basis points**." Simultaneously, the tenor basis—the difference in pricing between a swap referencing 1-month LIBOR and one referencing 3-month LIBOR—widened from near-zero to dramatic levels: "the difference between 1 month and 3 month Libor rates was in the order of one basis point up until September 2007, but since then has been as wide as **50 basis points**."

Practitioners who clung to the single-curve model found themselves systematically mispricing risks. A trader valuing a 6-month floating-rate note using a 3-month curve would overestimate its value; a risk manager hedging a 3-month exposure with a 1-month instrument would leave a massive "basis risk" exposure completely unhedged.

This chapter details the **multi-curve framework** that rose from the ashes of the single-curve world. This is not merely an optional refinement; it is the absolute standard for modern interest rate modeling. Building on the OIS discounting framework from Chapter 18, we explore:

1. **The Divorce of Discounting and Projection:** Why we now use one curve to value cash flows (Discounting) and completely separate curves to forecast them (Projection).
2. **Multi-Curve Valuation Mechanics:** How to price swaps and floating rate notes when the projection curve differs from the discount curve, including how projection curves are constructed as spreads to OIS.
3. **Tenor Basis Preview:** Why a 1-month rate and a 3-month rate are fundamentally different assets—setting up the detailed treatment in Chapter 20.
4. **The SOFR Transition:** The mechanics of Risk-Free Rates, including index types (Compound-in-Arrears, Term SOFR), observation conventions, and the credit-sensitivity gap that explains why banks resisted the transition.
5. **Risk Management Implications:** How "delta" splits into "discount risk" and "projection risk," and what instruments hedge each.

---

## 19.1 The "Single Curve" Fallacy

To understand why the market moved to multiple curves, we must first appreciate why the old model failed. In the pre-2007 world, a trader looking at a 3-month Forward Rate Agreement (FRA) and an Overnight Index Swap (OIS) saw them as two instruments telling the same story about the future path of interest rates.

### 19.1.1 The Pre-Crisis Simplicity

Consider a simplified market scenario where the overnight risk-free rate is flat at 2.00%. In a frictionless single-curve world, the 3-month forward rate should simply mathematically compound this 2.00% rate. The discount factor required to price an instrument at par would be consistent across all markets.

However, suppose quotes in the FRA market imply a 3-month forward rate of **3.40%**, significantly higher than the 2.00% implied by the overnight OIS market. This 140 basis point difference represents the **credit and liquidity premium** banks charge for unsecured term lending.

### 19.1.2 The Impossible Contradiction

If a quant tried to fit a **single** yield curve to this market, they would face an impossible contradiction:

* **The OIS Market** says the discount factor for a cash flow in 6 months ($T=0.50$) should be derived from the 2.00% rate: $P(0, 0.50) \approx 0.9900$.
* **The FRA Market** says the forward rate is 3.40%. For the forward rate to be 3.40% when the 3-month rate is 2.00%, the 6-month discount factor would need to be much lower, roughly $0.9866$, to "justify" the high forward rate.

One curve cannot simultaneously pass through $0.9900$ and $0.9866$ at $T=0.50$. The classic "no-arbitrage" relation $P(T_2) = P(T_1) / (1 + \tau F)$ holds mathematically, but it requires $P$ and $F$ to come from the *same* underlying economic reality. When OIS and LIBOR represent different credit risks, they cannot be described by a single function.

### 19.1.3 The Resolution

This contradiction forced the market to adopt a **dual-curve approach**: use the risk-free (OIS) curve to determine the *present value* of money (Discounting), and use specific market curves (LIBOR 3M, EURIBOR 6M, SOFR, etc.) to estimate *future rate fixings* (Projection).

As Andersen and Piterbarg explain: "it is now generally accepted that the Libor rate is no longer a good proxy for a discounting rate on collateralized trades."

> **Analogy: The Two Watches**
>
> In the old world, we had one watch. It told us the time (Discounting) and we used that same time to schedule our meetings (Projection).
>
> In the new world, we wear two watches:
> * **Watch 1 (OIS - The Gold Watch)**: This tells the "Value of Time." It determines the Present Value of money today. It is Risk-Free.
> * **Watch 2 (LIBOR/SOFR - The Weather Watch)**: This tells us "How much it will rain" (Forward Rates). It determines how much the floating leg pays.
>
> You naturally check Watch 2 to see *how much* to pay, and Watch 1 to see *what it's worth* today. Mixing them up leads to disaster.

---

## 19.2 The Multi-Curve Valuation Framework

In the modern framework, we define a "universal" discount curve and a family of "index-specific" projection curves. Andersen and Piterbarg formally define this as a **multi-index curve group**: "a collection $\{P(\cdot), P^1(\cdot), \ldots, P^K(\cdot)\}$ of the universal discounting curve and one index curve per tenor."

### 19.2.1 The Roles of the Curves

**1. The Discount Curve ($P_d$)**

This curve answers the question: *"What is a risk-free dollar at time $T$ worth today?"*

For collateralized derivatives (which constitute the vast majority of the market), the discount curve is constructed from **Overnight Index Swaps (OIS)**. Chapter 18 developed this curve in detail. Andersen and Piterbarg provide the rationale: "most inter-dealer transactions are collateralized under the International Swaps and Derivatives Association (ISDA) Master Agreement, with the rate paid on collateral being the Fed funds rate (for USD; Eonia and Sonia for Euro and GBP)."

Hull similarly explains that when a derivative is fully collateralized by cash earning the overnight rate, the funding cost for that position is the overnight rate. To avoid arbitrage, we must therefore discount at OIS.

**2. The Projection Curve ($P_k$)**

This curve answers the question: *"What will the floating index $k$ fix at in the future?"*

There is not just one projection curve. There is a specific curve for **3-Month LIBOR** (legacy), another for **6-Month EURIBOR**, another for **SOFR compounded**, and so on. These curves are often constructed from "pseudo-discount factors" $P_k(0,T)$, which are mathematical constructs designed to reproduce forward rates via the standard formula:

$$\boxed{F_k(0; T, T+\tau) = \frac{1}{\tau} \left( \frac{P_k(0, T)}{P_k(0, T+\tau)} - 1 \right)}$$

> **Important Distinction:** The pseudo-discount factors $P_k$ are not real discount factors—you cannot buy a zero-coupon bond with these prices. They are mathematical devices to ensure forwards price correctly. Andersen and Piterbarg refer to these as "index curves" precisely because they are linked to a specific observable index rather than to true discounting.

> **Desk Reality: Why the Terminology Matters**
>
> When a middle-office system shows you "forward rates from the 3M curve," it's computing $F = \frac{1}{\tau}\left(\frac{P^{3M}(T)}{P^{3M}(T+\tau)} - 1\right)$ using pseudo-discount factors. These forwards match FRA market quotes.
>
> **The trap:** Using these pseudo-discount factors for actual present-value calculations gives the wrong answer. Discounting at $P^{3M}$ when you should discount at $P^{OIS}$ systematically misprices collateralized swaps. Always ask your system: "Which curve discounts? Which curve projects?"

### 19.2.2 Valuation of a Floating Rate Note (FRN): The Par Floater Paradox

Let's apply this framework to a basic instrument: a Floating Rate Note (FRN) that pays a coupon linked to 3-Month LIBOR. In the single-curve world, a par-floater always prices exactly at par ($\$100$). In the multi-curve world, this intuitive rule breaks down completely.

**Example A: 1-Year Quarterly FRN**

Consider a 1-year FRN with quarterly coupons:
* **Notional:** $N = \$100$
* **Discount Curve ($P_d$):** Flat at 2.00% (OIS).
* **Projection Curve ($F_{3M}$):** The market forward rates for 3M LIBOR are 3.20%, 3.40%, 3.50%, and 3.60%.

To value this, we use the **Projection Curve** to estimate the cash flows and the **Discount Curve** to present-value them.

**Step 1: Project Cash Flows**

The coupons are $N \times 0.25 \times F_{3M}$:
* Q1: $100 \times 0.25 \times 3.20\% = \$0.800$
* Q2: $100 \times 0.25 \times 3.40\% = \$0.850$
* Q3: $100 \times 0.25 \times 3.50\% = \$0.875$
* Q4: $100 \times 0.25 \times 3.60\% = \$0.900$

**Step 2: Compute OIS Discount Factors**

At 2.00% flat (continuous compounding approximation for simplicity):
* $P_d(0.25) = e^{-0.02 \times 0.25} \approx 0.9950$
* $P_d(0.50) = e^{-0.02 \times 0.50} \approx 0.9901$
* $P_d(0.75) = e^{-0.02 \times 0.75} \approx 0.9851$
* $P_d(1.00) = e^{-0.02 \times 1.00} \approx 0.9802$

**Step 3: Discount to Present Value**

$$PV_{cpn} = 0.80(0.9950) + 0.85(0.9901) + 0.875(0.9851) + 0.90(0.9802) \approx \$3.38$$
$$PV_{prin} = 100(0.9802) \approx \$98.02$$

$$\boxed{\text{Total PV} = \$101.40}$$

**The Result:** The FRN prices at **101.40**, not 100.00.

Why? Because the coupons are "rich"—they are set by the higher risky curve (3.20%–3.60%)—but they are discounted by the lower risk-free curve (2.00%). The difference represents the present value of the credit premium embedded in the LIBOR rates.

> **The Par Floater Paradox:** A LIBOR-linked FRN discounted at OIS will trade *above* par when LIBOR > OIS. This violation of "par floater = par" is a defining feature of the multi-curve world.

### 19.2.3 Valuation of an Interest Rate Swap

The same logic applies to Interest Rate Swaps (IRS). The par swap rate $K_{par}$ is the fixed rate that equates the PV of the fixed leg to the PV of the floating leg.

$$\boxed{K_{par} = \frac{\sum_{i} \tau_i F_k(T_{i-1}, T_i) P_d(0, T_i)}{\sum_{i} \alpha_i P_d(0, T_i)}}$$

Notice the mix of curves: the numerator uses $F_k$ (Projection) to forecast floating payments and $P_d$ (Discount) to value them. The denominator uses $P_d$ (Discount) to value the fixed annuity.

**Example B: Par Swap Rate Calculation**

Using our numbers above, the "Projection PV" of the floating leg was $\$3.38$. The "Discount Annuity" (sum of OIS discount factors $\times$ 0.25) is:

$$\text{Annuity} = 0.25 \times (0.9950 + 0.9901 + 0.9851 + 0.9802) = 0.25 \times 3.9504 = 0.9876$$

$$K_{par} \approx \frac{3.38}{100 \times 0.9876} \approx 3.42\%$$

This par rate of **3.42%** is effectively a weighted average of the high projection forwards (3.2%–3.6%), not the low OIS rates.

### 19.2.4 Building the Projection Curve as a Spread to OIS

The projection curve is not constructed in isolation. In practice, it is built as a **spread over the OIS discount curve**. This approach ensures that perturbations to OIS affect discounting, while perturbations to basis instruments only affect projection—maintaining orthogonality in risk sensitivities.

Andersen and Piterbarg describe this spread-based construction:

$$\boxed{P^{(k)}(t) = P_d(t) \cdot e^{-\int_0^t \eta^{(k)}(s) \, ds}}$$

where $\eta^{(k)}(t)$ is the spread function for tenor $k$. This exponential spread representation is mathematically equivalent to saying that the projection curve's zero rates exceed OIS zero rates by the spread $\eta$.

**Why Spread-Based Construction?**

Andersen and Piterbarg explain the advantages: "The parameterization allows us to naturally aggregate 'similar' risks such as overall rate level risks, discounting risks, basis risks, while keeping different kinds separate for efficient risk management."

If we instead bootstrapped each curve independently from separate instruments, the curves might not be consistent with each other, and risk sensitivities would be entangled. The spread-based approach ensures:

1. **OIS curve perturbations** only affect discounting
2. **Basis swap perturbations** only affect the spread $\eta^{(k)}$
3. **Parallel shifts** in vanilla swaps affect both OIS and projection by construction

**Example C: Extracting the Spread**

Suppose we have:
* OIS discount factor: $P_d(0, 1) = 0.9500$
* 3M LIBOR projection pseudo-factor: $P^{3M}(0, 1) = 0.9450$

The implied spread:
$$e^{-\eta \times 1} = \frac{P^{3M}(1)}{P_d(1)} = \frac{0.9450}{0.9500} = 0.9947$$

$$\eta = -\ln(0.9947) = 0.53\% = 53 \text{ bps}$$

This 53bp spread represents the average OIS-LIBOR basis over the 1-year tenor.

> **Desk Reality: Why Traders Think in Spreads**
>
> When a trader says "3M LIBOR is trading 50 over OIS," they mean the spread $\eta \approx 50$ bps. Trading systems should decompose risk into:
> * **OIS delta:** Sensitivity to the base rate level (hedged with OIS swaps, Fed Funds futures)
> * **Basis delta:** Sensitivity to the OIS-LIBOR spread (hedged with basis swaps)
>
> If your system reports a single "interest rate delta," it's combining risks that require different hedge instruments. This mixing can hide large unhedged basis exposures.

### 19.2.5 Cross-Currency Context (Preview)

The multi-curve framework extends naturally to cross-currency settings. When valuing a EUR/USD cross-currency swap, we need:

* **USD OIS curve** for discounting USD cash flows
* **EUR OIS curve** (built via FX forwards and cross-currency basis swaps)
* **EUR EURIBOR projection curve** for forecasting EUR floating payments
* **USD SOFR projection curve** for forecasting USD floating payments

Covered Interest Parity (CIP) provides the linkage: FX forwards, OIS curves, and projection curves must be mutually consistent to avoid arbitrage. When they're not—which happened dramatically during 2008 and again in March 2020—the cross-currency basis widens.

**Chapter 21 develops this cross-currency framework fully.** Here we note only that the projection curve concept extends seamlessly: each currency has its own projection curves for each relevant tenor, all built as spreads to the appropriate discount curve.

---

## 19.3 The Tenor Basis: A Preview

So far we have distinguished between "Risk-Free" (OIS) and "Risky" (LIBOR/SOFR term). But "Risky" is not a monolith. In the post-crisis world, the market began to brutally distinguish between different *tenors* of risk.

### 19.3.1 Why 1-Month and 3-Month Rates Diverge

Pre-2007, a bank needing 3-month funding could comfortably assume that borrowing for 1 month and rolling the loan twice was roughly equivalent to borrowing for 3 months once. The market priced these nearly identically.

Andersen and Piterbarg document the historical baseline: "the difference between 1 month and 3 month Libor rates was in the order of one basis point up until September 2007."

During the liquidity crunch, this logic evaporated. A 3-month loan locks up liquidity for 90 days; a 1-month loan returns cash in 30 days. In a stressed market, that 60-day difference is valuable. Banks demanded a higher premium for the longer lock-up. Andersen and Piterbarg continue: "since then has been as wide as 50 basis points."

### 19.3.2 Economic Drivers of Tenor Basis

Andersen and Piterbarg explain that tenor basis arises "partly from credit considerations and partly liquidity considerations":

1. **Credit Horizon Risk:** A 6-month loan embeds more default risk than a 1-month loan to the same counterparty.
2. **Liquidity Preference:** "Banks have a natural desire to have longer-term deposits to better match their loan commitments." This creates structural demand for longer-dated funding.
3. **Market Segmentation:** Different tenors attract different participants with distinct supply/demand dynamics.

> **Visual: The Fan Chart**
>
> Imagine the yield curve as a fan opening up:
> * **Base Line (Bottom)**: OIS Curve (Risk Free)
> * **First Blade**: 1-Month Projection (Slightly higher)
> * **Second Blade**: 3-Month Projection (Higher still)
> * **Top Blade**: 6-Month Projection (Highest)
>
> The "Tenor Basis" is the space between the blades. In times of stress, the fan opens wide (basis widens). In calm times, the fan closes up (basis compresses).

### 19.3.3 Implications for Multi-Curve Construction

Because different tenors represent fundamentally different risk, each tenor requires its own projection curve:
* $P^{1M}(t)$ — projection curve for 1-month rates
* $P^{3M}(t)$ — projection curve for 3-month rates
* $P^{6M}(t)$ — projection curve for 6-month rates

These curves are linked by **basis swaps** (floating-for-floating swaps that exchange payments on different tenors). The basis swap spread calibrates the relationship between tenor curves.

**Chapter 20 develops the mathematics of tenor basis and basis swap bootstrapping in detail.** Here we emphasize only the conceptual point: you cannot use the 3-month curve to forecast 1-month or 6-month rates without introducing systematic errors.

---

## 19.4 The Transition to SOFR

The principles of the multi-curve framework—separating discounting from projection—remain perfectly valid in the new world of Risk-Free Rates (RFRs) like SOFR (Secured Overnight Financing Rate). However, SOFR introduces new mechanical complexity: it is **backward-looking**.

### 19.4.1 Forward-Looking vs Backward-Looking Rates

Hull explains the key difference: "LIBOR rates are forward looking. They are determined at the beginning of the period to which they will apply. The new reference rates are backward looking. The rate applicable to a particular period is not known until the end of the period when all the relevant overnight rates have been observed."

> **Analogy: Weather Forecast vs Thermometer**
>
> * **LIBOR** is like a weather forecast: At the start of the week, someone predicts "It will average 75°F this week." You know upfront.
> * **SOFR** is like reading a thermometer: You record the temperature each day, and at week's end you calculate "This week averaged 74.3°F." You only know afterward.
>
> The operational challenge: How do you make a payment on time if you don't know the amount until the period ends?

### 19.4.2 SOFR Index Types

The SOFR market has developed multiple index types to address different operational and hedging needs. Understanding these is essential for middle-office professionals who see them on trade confirmations.

**1. Compound-in-Arrears (Standard)**

This is the most common convention for SOFR derivatives. Daily overnight rates are compounded over the accrual period:

$$\boxed{\text{Compounded Rate} = \left[\prod_{i=1}^{n}(1 + r_i \cdot \hat{d}_i) - 1\right] \times \frac{360}{D}}$$

where $r_i$ is the overnight SOFR on day $i$, $\hat{d}_i = d_i/360$ is the day fraction, $d_i$ is the number of days that rate applies (usually 1, but 3 over weekends), and $D = \sum_i d_i$ is the total days in the period.

Hull confirms this formula, noting that "longer rates such as three-month rates, six-month rates, or one-year rates can be determined from overnight rates by compounding them daily."

**Example D: SOFR Compound-in-Arrears Calculation**

Calculate the compounded SOFR rate for a 7-day period (Monday through Sunday):

| Day | SOFR Rate | Days Applied | Growth Factor |
|-----|-----------|--------------|---------------|
| Mon | 5.30% | 1 | $1 + 0.0530 \times \frac{1}{360} = 1.0001472$ |
| Tue | 5.32% | 1 | $1 + 0.0532 \times \frac{1}{360} = 1.0001478$ |
| Wed | 5.31% | 1 | $1 + 0.0531 \times \frac{1}{360} = 1.0001475$ |
| Thu | 5.29% | 1 | $1 + 0.0529 \times \frac{1}{360} = 1.0001469$ |
| Fri | 5.30% | 3 | $1 + 0.0530 \times \frac{3}{360} = 1.0004417$ |

**Step 1:** Compound the daily factors:
$$\prod = 1.0001472 \times 1.0001478 \times 1.0001475 \times 1.0001469 \times 1.0004417 = 1.001031$$

**Step 2:** Annualize:
$$\text{Compounded Rate} = (1.001031 - 1) \times \frac{360}{7} = 5.303\%$$

**Sanity check:** The compounded rate (5.303%) is slightly higher than the arithmetic average of daily rates (~5.30%) because compounding adds a small convexity pickup. ✓

**2. Daily Simple SOFR**

For some lending products, particularly bilateral loans, daily rates are simply averaged (not compounded):

$$\text{Simple Average} = \frac{\sum_i r_i \cdot d_i}{D}$$

This convention is operationally simpler but mathematically less precise than compounding.

**3. SOFR Averages (30-day, 90-day, 180-day)**

The New York Fed publishes pre-calculated rolling averages:
* **SOFR 30-day Average:** Compounded SOFR over the prior 30 days
* **SOFR 90-day Average:** Compounded SOFR over the prior 90 days

These are useful for products that reference a single published rate rather than computing compounding internally.

**4. Term SOFR (CME)**

CME publishes **Term SOFR** rates that are forward-looking—derived from SOFR futures prices. These behave like LIBOR: known at the start of the period.

| Tenor | Description |
|-------|-------------|
| 1-month Term SOFR | Forward-looking 30-day rate |
| 3-month Term SOFR | Forward-looking 90-day rate |
| 6-month Term SOFR | Forward-looking 180-day rate |

> **Desk Reality: Why Term SOFR Exists**
>
> Corporate treasurers and loan markets strongly preferred a LIBOR-like forward-looking rate. "We need to know what we owe at the start of the period for budgeting and cash management." Term SOFR fills this gap.
>
> However, Term SOFR is not freely usable in all products—see Section 19.4.5 for regulatory restrictions.

### 19.4.3 Observation Conventions

The challenge with backward-looking rates is that the payment amount is not known until the period ends—leaving no time for operational processing. Markets have developed several conventions to address this:

**1. Payment Delay**

The simplest approach: Payment occurs 2-5 business days after the accrual period ends.

* **Mechanism:** Full compounding through period end; payment delayed
* **Trade-off:** Slight timing mismatch; operationally simple
* **Example:** Period ends June 30; payment occurs July 2

**2. Lookback (with or without Observation Shift)**

The observation period is shifted backward relative to the accrual period.

* **Lookback without Shift:** Observe rates from 5 days earlier, but apply them to the actual accrual schedule
* **Lookback with Observation Shift:** Shift both observation *and* weighting (more complex)
* **Trade-off:** Payment known in advance; slight hedging mismatch
* **Example:** For accrual March 1–31, observe SOFR from Feb 24–March 26

**3. Lockout (Rate Freeze)**

The rate is frozen for the last few days of the period, using the penultimate observation.

* **Mechanism:** Last 2-5 days use the same rate as the lockout start date
* **Trade-off:** Payment known in advance; rate approximation
* **Example:** Period ends June 30; June 29-30 both use June 28's rate

> **Practitioner Note: Convention Basis Risk**
>
> Different conventions create small but systematic differences. If you have a SOFR loan with 5-day lookback and hedge with a swap using 2-day payment delay, there's a "convention basis" mismatch. Over many periods, this can compound into material P&L differences.
>
> I'm not sure about exact magnitudes—these vary by rate environment and are still being studied as the market matures.

### 19.4.4 The Credit-Sensitivity Gap: Why Banks Resisted SOFR

A crucial economic concept that explains the contentious nature of the LIBOR transition is the **credit-sensitivity gap**.

LIBOR included a credit spread—it reflected the cost of unsecured lending between banks. When market stress increased, LIBOR rose (banks charged more to lend to each other). This meant that banks holding LIBOR-linked assets saw their income rise when their funding costs rose. There was a natural hedge.

SOFR, being a secured overnight rate, does not include credit spread. In a stress scenario:
* **Bank funding costs rise** (banks still borrow unsecured; counterparties demand more)
* **SOFR-linked asset income stays flat** (SOFR is secured, unaffected by bank credit)

This creates a **funding mismatch**: banks' assets don't re-price when their liabilities do.

> **Analogy: Insurance Premium That Doesn't Rise**
>
> Imagine you're a homeowner. Your insurance premium rises every time there's a hurricane (that's LIBOR—you pay more when risk rises, but your income from renting the house also rises because tenants pay LIBOR-linked rent).
>
> Now imagine your rent income is locked to a "hurricane-free rate" (that's SOFR). When hurricanes hit:
> * Your insurance costs spike
> * Your rental income stays flat
> * You have a cash flow crisis
>
> This is the bank's problem with SOFR. It's economically "safer" for the system, but it shifts risk onto bank balance sheets.

### 19.4.5 Term SOFR Regulatory Restrictions

In response to concerns that Term SOFR might become "new LIBOR"—a heavily used benchmark susceptible to manipulation and systemic risk—the Alternative Reference Rates Committee (ARRC) imposed restrictions on its use.

> **Practitioner Note (Priority 2 — not sourced from books/):**
>
> ARRC's Term SOFR guidance restricts its use in certain products:
> * **Permitted:** Business loans, commercial real estate, FRNs for end-users
> * **Restricted:** Interdealer derivatives markets (to prevent Term SOFR from becoming dominant and creating systemic risk similar to LIBOR)
>
> **Hedging Implication:** A bank with a Term SOFR loan book cannot directly hedge with Term SOFR swaps in the interdealer market. They must hedge with Compound SOFR swaps, creating **Term-Compound basis risk**.
>
> I'm not sure about the exact current status of these restrictions—regulatory guidance may evolve. Verify with current ARRC documentation.

**Example E: Term SOFR Hedge Basis Risk**

A bank has $1 billion in Term SOFR loans and hedges with Compound SOFR swaps. If the Term SOFR / Compound SOFR basis widens by 5 bps:

$$\text{Basis P&L Impact} \approx \$1B \times 0.0005 \times \text{Average Duration} \approx \$1B \times 0.0005 \times 3 = \$1.5\text{mm}$$

This is unhedged risk that exists purely because of the regulatory constraint on hedge instruments.

### 19.4.6 Credit-Sensitive Rate Alternatives

In response to the credit-sensitivity gap, some institutions explored credit-sensitive alternatives to SOFR.

> **Practitioner Note (Priority 2):**
>
> * **BSBY (Bloomberg Short-Term Bank Yield Index):** A credit-sensitive rate derived from bank funding transactions. Gained some traction in lending markets but faced regulatory pushback.
> * **Ameribor:** Based on unsecured overnight loans between small and mid-size banks. Used by regional banks concerned about SOFR's disconnect from their funding costs.
>
> As of late 2023, Bloomberg announced it would discontinue BSBY, and the regulatory consensus has firmly settled on SOFR as the primary USD benchmark. Credit-sensitive rates exist but are not recommended for broad use.
>
> The current market consensus is clear: SOFR has won. But understanding *why* alternatives were explored illuminates the economic tensions in the transition.

### 19.4.7 The Return of the Single Curve?

Interestingly, for a standard SOFR OIS swap (where we pay fixed and receive compounded SOFR), the "Projection" curve and the "Discount" curve are conceptually the same (both are SOFR). In this specific corner of the market, the "single curve" world has effectively returned.

However, as soon as we trade a **BSBY** swap, or a **Term SOFR** swap, or a legacy **LIBOR** instrument, or involve **foreign currency** cash flows, the multi-curve distinction comes roaring back. The modern desk must handle both regimes simultaneously.

---

## 19.5 Managing Risk in a Multi-Curve World

The shift to multiple curves complicates risk management. The simple question "what is my delta?" now has two answers.

### 19.5.1 Discount Risk vs. Projection Risk

Consider the 1-year Swap from our earlier example (Receive Fix 3.42% / Pay Float).

* **Discount Risk (OIS Delta):** If the **OIS curve** shifts up by 1 basis point, the present value of the fixed leg decreases (standard duration effect). The floating leg PV also changes slightly because the *weights* on the cash flows change.
* **Projection Risk (Index Delta):** If the **3M LIBOR projection curve** shifts up by 1 basis point, the *amount* of the floating coupons increases. This is a direct hit to the floating leg value.

**Example F: Sensitivity Comparison**

For our sample trade:
* **Bump OIS +1bp:** $\Delta PV \approx 0$ (The trade is approximately par, so sensitivity to the discount rate is minimal)
* **Bump 3M LIBOR +1bp:** $\Delta PV \approx -10$ bps (We are paying floating, so higher rates hurt us directly)

A risk manager who lumps these together into a single "Interest Rate Delta" would miss the fact that the desk is essentially neutral to Fed Funds rate moves but heavily short the 3M LIBOR level. These risks must be hedged with different instruments.

### 19.5.2 Hedge Instrument Mapping

| Risk Type | Description | Hedge Instruments |
|-----------|-------------|-------------------|
| **OIS/Discount Risk** | Sensitivity to collateral rate | OIS swaps, Fed Funds futures, SOFR futures |
| **Projection Risk (SOFR)** | Sensitivity to SOFR curve | SOFR OIS swaps, SOFR futures |
| **Projection Risk (Term)** | Sensitivity to term rates | Term rate derivatives (restricted) |
| **Tenor Basis Risk** | Sensitivity to spread between tenors | Basis swaps (3M vs 6M, etc.) |
| **OIS-Index Basis** | Sensitivity to OIS-LIBOR/SOFR spread | OIS vs 3M basis swaps |

### 19.5.3 Orthogonality in Risk Sensitivities

A robust curve construction strives for **orthogonality**. Andersen and Piterbarg describe the risk decomposition:

* "Perturbations to instruments used in building the base index curve... define risk sensitivities to the overall levels of interest rates."
* "Perturbations to funding instruments define sensitivities to discounting."
* "Perturbations to basis swap spreads for $L^k$-versus-$L^1$ floating-floating basis swaps define basis risk."

This parameterization "allows us to naturally aggregate 'similar' risks such as overall rate level risks, discounting risks, basis risks, while keeping different kinds separate for efficient risk management."

### 19.5.4 Basis Trading

> **Desk Reality: What is "Basis Trading"?**
>
> When traders say "I'm long basis," they typically mean they expect the spread between a risky rate (LIBOR, SOFR term) and the risk-free rate (OIS) to widen. This is a bet on credit stress or funding pressure.
>
> **Long basis = expect spread to widen = expect bank stress**
> **Short basis = expect spread to compress = expect calm markets**
>
> The instruments:
> * OIS vs 3M SOFR basis swap: Pay OIS, receive SOFR → long the SOFR-OIS spread
> * 3M vs 6M basis swap: Pay 3M, receive 6M → long the tenor basis
>
> Basis trading was extremely profitable during 2008 and March 2020 when spreads exploded.

---

## 19.6 Practical Notes

### 19.6.1 Curve Construction Hierarchy

Build curves in order of liquidity:
1. **OIS (Discount):** From OIS swaps and Fed Funds/SOFR futures (Chapter 18)
2. **Base Projection (SOFR or legacy 3M LIBOR):** From vanilla fixed-float swaps
3. **Secondary Tenors (1M, 6M):** From basis swaps as spreads to the base (Chapter 20)

### 19.6.2 Common Pitfalls

| Pitfall | Description | Consequence |
|---------|-------------|-------------|
| **Wrong discount curve** | Legacy systems may still discount at LIBOR | Systematic mispricing; P&L breaks |
| **Ignoring tenor basis** | Hedging 6M exposure with 3M instruments | Residual basis risk |
| **Confusing pseudo-DF** | Treating projection curve $P_k$ as real prices | Wrong present values |
| **Convention mismatch** | Mixing Term SOFR and Compound SOFR | Hedge basis risk |
| **Missing credit gap** | Not understanding SOFR vs funding cost | Unexpected P&L in stress |

### 19.6.3 Implementation Checklist

* **Repricing Check:** After building curves, reprice every input instrument. Errors should be $< 10^{-10}$.
* **Orthogonality Test:** Bump basis spreads and verify only projection curves move, not discount.
* **Convention Match:** Ensure day counts (ACT/360 vs ACT/365) match the specific index.
* **SOFR Convention Match:** Verify lookback, lockout, or payment delay matches the instrument.

---

## Summary

1. **Discount at the Collateral Rate:** Value all cash flows using the OIS curve (assuming standard CSA). This is the foundational principle from Chapter 18.

2. **Project at the Index Rate:** Forecast floating coupons using the specific curve for that index (3M SOFR, 6M EURIBOR, etc.). These curves are built as spreads to OIS.

3. **Par Floaters Aren't Par:** A LIBOR-linked FRN discounted at OIS will trade above par when LIBOR > OIS. This is mathematically inevitable in the multi-curve framework.

4. **SOFR is Backward-Looking:** Unlike LIBOR, SOFR is computed from realized overnight rates. Multiple index types (Compound-in-Arrears, Daily Simple, Term SOFR) address operational needs.

5. **Credit-Sensitivity Gap:** SOFR doesn't rise when bank funding costs rise, creating a mismatch for bank balance sheets. This explains why the LIBOR transition was contentious.

6. **Term SOFR Restrictions:** Regulatory guidance limits Term SOFR use in interdealer derivatives, creating basis risk when hedging Term SOFR loans.

7. **Decompose Your Risk:** Always distinguish between sensitivity to the discount curve (OIS delta), sensitivity to the projection curve (index delta), and sensitivity to the spread between them (basis delta).

8. **Preview of Tenor Basis:** Different funding tenors (1M, 3M, 6M) represent different credit/liquidity risks and require separate projection curves—developed fully in Chapter 20.

The multi-curve framework is more complex than the single-curve world. But it is also more honest. It forces us to confront the reality that liquidity has a price, credit has a price, and in financial markets, not all dollars are created equal.

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **Discount Curve** | The OIS curve used to present-value all cash flows | Reflects the true cost of funding collateralized trades |
| **Projection Curve** | Tenor-specific curve (e.g., 3M SOFR) used to forecast floating fixings | Ensures forward rates match market FRA/swap quotes |
| **Pseudo-Discount Factor** | $P_k(T)$ values that reproduce forward rates via standard formula | Not real prices—mathematical machinery for projection |
| **Multi-Index Curve Group** | Collection of one discount + multiple projection curves | The only consistent framework for post-crisis pricing |
| **Spread-Based Construction** | Building projection curve as $P^{(k)} = P_d \cdot e^{-\int \eta}$ | Ensures orthogonal risk decomposition |
| **Tenor Basis** | Spread between forward rates of different tenors | Represents credit/liquidity risk differences by funding horizon |
| **Par Floater Paradox** | LIBOR FRN trades above par when LIBOR > OIS | Classic manifestation of the two-curve divergence |
| **SOFR Compound-in-Arrears** | Standard SOFR: daily rates compounded over period | Backward-looking; not known until period end |
| **Term SOFR** | Forward-looking SOFR from futures | Like LIBOR; regulatory restrictions apply |
| **Credit-Sensitivity Gap** | SOFR doesn't rise with bank funding costs | Explains bank resistance to SOFR transition |
| **Orthogonality** | Discount and projection risks are independent | Enables clean hedging and risk attribution |

---

## Notation for This Chapter

| Symbol | Definition |
|--------|------------|
| $P_d(t,T)$ | Discount factor from the OIS/collateral curve |
| $P_k(t,T)$ or $P^{(k)}(t,T)$ | Pseudo-discount factor for projection curve $k$ |
| $F_k(t; T_1, T_2)$ | Forward rate for tenor $k$ over $[T_1, T_2]$ |
| $\eta^{(k)}(t)$ | Spread function: projection curve $k$ vs OIS |
| $K_{par}$ | Par swap rate (makes swap PV = 0 at inception) |
| $\tau, \alpha$ | Year fractions for floating and fixed legs |
| $r_i$ | Overnight SOFR rate on day $i$ |
| $d_i$ | Number of days that rate $r_i$ applies |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | Why did the single-curve framework fail after 2007? | The spread between OIS and LIBOR widened to 275 bps, revealing that they represent different credit risks and cannot be described by a single curve. |
| 2 | What is a projection curve used for? | Forecasting future floating rate fixings for a specific index (e.g., 3M SOFR). |
| 3 | What is a pseudo-discount factor $P_k(T)$? | A mathematical construct that reproduces forward rates via $F = \frac{1}{\tau}(P_k(T_1)/P_k(T_2) - 1)$. Not a real bond price. |
| 4 | What is a "multi-index curve group"? | A collection of one discount curve and multiple projection curves, one per tenor. |
| 5 | Why does a LIBOR FRN not price at par when discounted at OIS? | The coupons are set at the higher LIBOR rate but discounted at the lower OIS rate, creating PV > par. |
| 6 | What is spread-based curve construction? | Building projection curves as $P^{(k)} = P_d \cdot e^{-\int \eta}$ to ensure orthogonal risk decomposition. |
| 7 | What is tenor basis? | The spread between forward rates of different tenors (e.g., 1M vs 3M), reflecting credit/liquidity differences. |
| 8 | How wide did the 1M–3M tenor basis get during the 2008 crisis? | Up to 50 basis points (from ~1 bp pre-crisis). |
| 9 | What is the key difference between LIBOR and SOFR? | LIBOR is forward-looking (set at period start); SOFR is backward-looking (compounded over period). |
| 10 | What is "compound-in-arrears"? | The standard SOFR convention: daily overnight rates compounded over the accrual period. |
| 11 | What is Term SOFR? | A forward-looking rate derived from SOFR futures, published by CME. Known at period start like LIBOR. |
| 12 | Why are there restrictions on Term SOFR use? | To prevent it from becoming "new LIBOR"—a dominant benchmark with systemic risk and manipulation potential. |
| 13 | What is the "credit-sensitivity gap"? | SOFR doesn't rise when bank funding costs rise, creating a mismatch between assets and liabilities in stress. |
| 14 | What is "discount risk" vs "projection risk"? | Discount risk = sensitivity to OIS curve; Projection risk = sensitivity to the specific index curve. |
| 15 | What instruments hedge projection risk? | SOFR futures, FRAs, fixed-float swaps on the relevant index. |
| 16 | What instruments hedge basis risk? | Basis swaps (floating-for-floating). |
| 17 | What is "basis trading"? | Betting on the spread between different rate indices (e.g., OIS vs SOFR, 3M vs 6M). |
| 18 | When does the single-curve framework still apply? | For SOFR OIS swaps, where discount and projection are both the SOFR curve. |

---

## Mini Problem Set

### Problem 1 (Basic)

You observe the following for a 6-month period:
- OIS discount factor: $P_d(0, 0.5) = 0.9900$
- 3M LIBOR projection pseudo-factors: $P_{3M}(0, 0.25) = 0.9940$, $P_{3M}(0, 0.5) = 0.9870$

Calculate the 3M forward rate for the period $[0.25, 0.5]$.

**Solution:**
$$F = \frac{1}{0.25}\left(\frac{0.9940}{0.9870} - 1\right) = \frac{1}{0.25}(1.00709 - 1) = 2.84\%$$

---

### Problem 2 (Basic)

An FRN pays 3M LIBOR quarterly for 1 year. The four quarterly 3M forwards are 3.0%, 3.1%, 3.2%, 3.3%. The OIS curve is flat at 2.5%. Calculate the FRN's present value per $100 notional.

**Solution:**
OIS discount factors (continuous compounding, flat curve):
- $P(0.25) = e^{-0.025 \times 0.25} = 0.9938$
- $P(0.5) = e^{-0.025 \times 0.50} = 0.9876$
- $P(0.75) = e^{-0.025 \times 0.75} = 0.9815$
- $P(1.0) = e^{-0.025 \times 1.00} = 0.9753$

Coupon PVs:
- Q1: $100 \times 0.25 \times 0.030 \times 0.9938 = 0.744$
- Q2: $100 \times 0.25 \times 0.031 \times 0.9876 = 0.766$
- Q3: $100 \times 0.25 \times 0.032 \times 0.9815 = 0.785$
- Q4: $100 \times 0.25 \times 0.033 \times 0.9753 = 0.805$

Principal PV: $100 \times 0.9753 = 97.53$

**Total PV = 0.744 + 0.766 + 0.785 + 0.805 + 97.53 = $100.63$**

The FRN trades above par because LIBOR > OIS.

---

### Problem 3 (Intermediate)

Given OIS discount factor $P_d(0, 1) = 0.9500$ and 3M projection pseudo-factor $P^{3M}(0, 1) = 0.9430$, calculate the implied OIS-LIBOR spread.

**Solution:**
$$\frac{P^{3M}(1)}{P_d(1)} = \frac{0.9430}{0.9500} = 0.9926$$

$$e^{-\eta \times 1} = 0.9926 \implies \eta = -\ln(0.9926) = 0.74\% = 74 \text{ bps}$$

---

### Problem 4 (Intermediate)

You hold a swap: receive fixed 3.50%, pay 3M LIBOR.
- DV01 to OIS curve: +$8,500 per 1 bp
- DV01 to 3M LIBOR curve: -$9,200 per 1 bp

If OIS rises 5 bps and 3M LIBOR rises 8 bps, what is your approximate P&L?

**Solution:**
$$\Delta PV = (+8,500)(+5) + (-9,200)(+8) = 42,500 - 73,600 = -\$31,100$$

You lose $31,100. The projection risk dominates because 3M LIBOR moved more.

---

### Problem 5 (Intermediate — SOFR Compounding)

Calculate the compounded SOFR rate for a 7-day period with the following daily rates:

| Day | SOFR Rate | Days Applied |
|-----|-----------|--------------|
| Mon | 4.80% | 1 |
| Tue | 4.82% | 1 |
| Wed | 4.79% | 1 |
| Thu | 4.81% | 1 |
| Fri | 4.80% | 3 |

**Solution:**

Daily factors:
- Mon: $1 + 0.048 \times \frac{1}{360} = 1.0001333$
- Tue: $1 + 0.0482 \times \frac{1}{360} = 1.0001339$
- Wed: $1 + 0.0479 \times \frac{1}{360} = 1.0001331$
- Thu: $1 + 0.0481 \times \frac{1}{360} = 1.0001336$
- Fri: $1 + 0.048 \times \frac{3}{360} = 1.0004000$

Compound:
$$\prod = 1.0001333 \times 1.0001339 \times 1.0001331 \times 1.0001336 \times 1.0004000 = 1.0009342$$

Annualize:
$$\text{Rate} = (1.0009342 - 1) \times \frac{360}{7} = 4.803\%$$

---

### Problem 6 (Advanced)

Explain mathematically why a floating rate note paying index $k$ does not trade at par when discounted at OIS.

**Solution Sketch:**

In the single-curve world, the PV of the floating leg telescopes:
$$PV_{float} = \sum_i \tau_i F P(T_i) = \sum_i \left[\frac{P(T_{i-1}) - P(T_i)}{P(T_i)}\right] P(T_i) = \sum_i [P(T_{i-1}) - P(T_i)] = 1 - P(T_n)$$

Adding principal: $PV_{FRN} = 1 - P(T_n) + P(T_n) = 1$ (par).

In multi-curve: $F_k$ is computed from $P_k$, but discounting uses $P_d$. The telescoping fails because:
$$PV_{float} = \sum_i \tau_i F_k P_d(T_i) \neq 1 - P_d(T_n)$$

When $F_k > F_d$ (projection rate > discount rate), the coupons are "rich" relative to the discount rate, so $PV > 1$.

---

### Problem 7 (Advanced)

A bank has $500mm in Term SOFR loans (3-year average life) and hedges with Compound SOFR swaps. The Term SOFR / Compound SOFR basis widens by 3 bps. Estimate the unhedged P&L impact.

**Solution:**
$$\text{Basis P&L} \approx \$500\text{mm} \times 0.0003 \times 3 \text{ years} = \$450,000$$

This is an unhedged loss if the bank is receiving Term SOFR (on loans) and paying Compound SOFR (on hedges) — the basis widening means their received rate rises less than their paid rate.

---

### Problem 8 (Advanced)

The OIS curve is flat at 2.00%. The 3M LIBOR projection curve implies forwards averaging 2.80%. A trader prices a 5-year swap using the single (LIBOR) curve for both discounting and projection. What error does this introduce, and in which direction?

**Solution Sketch:**

Using LIBOR for discounting (at 2.80%) instead of OIS (at 2.00%) means:
- Discount factors are too low (higher rate → lower DF)
- PVs of all legs are understated

For a receiver swap (receive fixed, pay floating):
- The fixed leg PV is understated (discounted too heavily)
- The floating leg PV is also understated, but for a par swap they're approximately equal

The key error is in the **fixed leg valuation**. The receiver overstates their position's value when marking to the correct (OIS) discount curve, because they used too high a discount rate.

For a 5-year swap with notional $100M and an 80 bp OIS-LIBOR spread, the mispricing of the fixed leg annuity alone is approximately:
$$\Delta \text{Annuity} \approx \text{Duration} \times \text{Spread} \times \text{Notional} \approx 4.5 \times 0.008 \times \$100M = \$3.6M$$

This is a rough estimate; actual mispricing depends on the specific cash flow structure.

---

## Source Map

### (A) Book-Verified Facts

| Fact | Source |
|------|--------|
| Fed Funds-LIBOR spread widened to 275 bps post-2007 | Andersen & Piterbarg Vol 1, §6.5.3 |
| 1M-3M tenor basis widened from ~1 bp to 50 bps | Andersen & Piterbarg Vol 1, §6.5.3 |
| OIS discounting for collateralized trades | Andersen & Piterbarg Vol 1, §6.5.3; Hull Ch 4, Ch 7 |
| Definition of multi-index curve group | Andersen & Piterbarg Vol 1, §6.5.3 |
| Forward rate from index curve formula (6.48) | Andersen & Piterbarg Vol 1, §6.5.3 |
| Swap valuation formula mixing curves (6.47) | Andersen & Piterbarg Vol 1, §6.5.3 |
| Spread-based curve construction: $P^k = P_d \cdot e^{-\int \eta}$ | Andersen & Piterbarg Vol 1, §6.5.3 |
| Risk decomposition: discount vs projection vs basis | Andersen & Piterbarg Vol 1, §6.5.3 |
| SOFR as backward-looking compounded rate | Hull Ch 4, Ch 6 |
| SOFR compounding formula | Hull Ch 4 |
| LIBOR forward-looking vs RFR backward-looking | Hull Ch 4 |
| Tenor basis driven by credit and liquidity | Andersen & Piterbarg Vol 1, §6.5.3 |
| Pseudo-discount factors called "index curves" | Andersen & Piterbarg Vol 1, §6.5.3 |

### (B) Claude-Extended Content

| Content | Basis |
|---------|-------|
| "Two Watches" analogy | Conceptual framing of discount vs projection roles |
| "Weather Forecast vs Thermometer" analogy | Conceptual framing of forward vs backward-looking |
| Credit-sensitivity gap explanation | Extends from A&P's credit/liquidity discussion to bank funding economics |
| Term SOFR regulatory restrictions | Post-2020 ARRC guidance (Priority 2 knowledge) |
| BSBY / Ameribor discussion | Post-2020 market developments (Priority 2 knowledge) |
| SOFR observation conventions (lookback, lockout, payment delay) | Standard market practice (Priority 2) |
| Basis trading desk reality | General trading desk practice |
| Hedge instrument mapping table | Synthesized from A&P risk decomposition |
| Term SOFR vs Compound SOFR basis risk example | Derived from regulatory constraint implications |

### (C) Reasoned Inference (Derived from A and B)

| Inference | Derivation |
|-----------|------------|
| Par Floater Paradox: FRN ≠ 100 | Derived from the fact that $P_k \neq P_d$ breaks the telescoping sum identity |
| Single curve is inconsistent | Derived from the mathematical impossibility of fitting one curve to both OIS and FRA markets |
| Par swap formula mixes curves | Direct algebraic consequence of projecting at $F_k$ and discounting at $P_d$ |
| Spread extraction formula | Rearrangement of Andersen & Piterbarg's spread-based construction |

### (D) Flagged Uncertainties

| Topic | Uncertainty |
|-------|-------------|
| **Term SOFR regulatory status** | I'm not sure about current ARRC guidance — it may have evolved since initial restrictions. Verify with current documentation. |
| **BSBY discontinuation details** | The exact timeline and transition arrangements should be verified with Bloomberg. |
| **SOFR observation convention magnitudes** | Convention basis risk exists but exact magnitudes vary by environment and are still being studied. |
| **Credit-sensitive rate viability** | The regulatory landscape for alternatives like Ameribor may continue to evolve. |
