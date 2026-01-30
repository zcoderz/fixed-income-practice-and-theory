# Chapter 34: XVA Overview — CVA, DVA, FVA, and the New Derivatives Valuation Landscape

---

## Introduction

When a derivatives desk marks a swap to market, what is the "true" value? Before 2008, the answer seemed obvious: discount expected cashflows at LIBOR, and you had your price. But the credit crisis shattered this simplicity. Counterparties default. Collateral earns interest. Funding has costs. Regulators demand capital. Each of these realities requires an adjustment to the "clean" price—and collectively, these adjustments have spawned an alphabet soup: CVA, DVA, FVA, MVA, KVA. Welcome to the world of XVAs.

Hull (OFD Chapter 9) defines the starting point precisely: "Most of this book is concerned with determining the no-default value of derivatives, that is, the value assuming that neither of the two sides will default. CVA and DVA are adjustments to the no-default value reflecting the possibility of a default by one of the two sides." But as we will see, the adjustments do not stop at credit. Funding costs, margin requirements, and capital charges each demand their own accounting—creating what practitioners call the XVA "stack" that sits atop the clean valuation.

> **Desk Reality: XVA as Manufacturing Costs**
>
> Think of XVA charges like manufacturing costs in a factory:
> - **CVA:** Insurance premium against counterparty default (like fire insurance on your plant)
> - **DVA:** Benefit from the possibility you might default (controversial—see Section 34.4)
> - **FVA:** Cost of borrowing from Treasury to fund the trade (like raw material costs)
> - **MVA:** Cost of posting initial margin (like working capital tied up in inventory)
> - **KVA:** Rent for using the bank's balance sheet (like factory floor rental)
>
> When Sales quotes a client "mid-market," the trader protests: "You forgot my costs!"
> A $1M NPV trade might net only $700K after XVA charges. The desk's real profitability depends on accounting for these manufacturing costs at trade inception.

This framing—XVA as manufacturing costs—is essential for practitioners. While academics debate whether FVA and KVA "should" exist in theory, the operational reality is that these charges affect deal economics, compensation, and even whether trades get done. A corporate treasurer who ignores the dealer's CVA charge will be puzzled by the quote. A risk manager who ignores wrong-way risk will underestimate tail exposures. A trader who doesn't understand KVA will lose money on every "profitable" trade that consumes too much capital.

This chapter provides a conceptual and practical overview of XVAs, connecting to the exposure metrics from Chapter 32 and the collateral discounting principles from Chapter 33. We cover:

1. **CVA: Credit Value Adjustment** — the cost of counterparty default
2. **DVA: Debit Value Adjustment** — the controversial "benefit" from your own potential default
3. **FVA: Funding Value Adjustment** — the cost (or benefit) of funding derivative positions, including the academic controversy
4. **MVA and KVA: Margin and Capital Adjustments** — newer additions to the XVA stack, with KVA's connection to Basel III capital requirements
5. **Wrong-way risk** — when exposure and default become correlated
6. **The XVA desk** — how banks organize and charge for these adjustments
7. **Practical implementation** — what goes into an XVA calculation

Why should you care? Because XVAs are not just accounting entries—they affect deal economics, hedge ratios, and even whether trades get done. Understanding XVAs is now table stakes for anyone in derivatives markets.

---

## 34.1 The Clean Valuation Baseline

Before diving into adjustments, we must establish what we are adjusting *from*. This section defines the no-default value that serves as the baseline for all XVA calculations.

### 34.1.1 No-Default Value

The **no-default value** of a derivatives portfolio, which Hull denotes $f_{\text{nd}}$, is "the value of the portfolio assuming that neither side will default." This is the output of standard derivatives pricing models—Black-Scholes-Merton for options, discounted expected cashflows for swaps—with no credit considerations.

For collateralized trades, Chapter 33 established that the appropriate discounting rate is the collateral remuneration rate (typically OIS). This gives us the **clean OIS-discounted value**. For uncollateralized trades, the discounting question becomes more complex, as funding and credit considerations enter.

### 34.1.2 The Adjustment Stack

Once we account for real-world frictions, the all-in value becomes:

$$\boxed{V_{\text{all-in}} = f_{\text{nd}} - \text{CVA} + \text{DVA} - \text{FVA} - \text{MVA} - \text{KVA} + \cdots}$$

Each term has economic meaning and a definite sign convention:

| Adjustment | Sign in Formula | Economic Interpretation |
|------------|-----------------|------------------------|
| **CVA** | Subtracted (reduces value) | Cost of counterparty default |
| **DVA** | Added (increases value) | "Benefit" from own potential default |
| **FVA** | Subtracted if net cost | Funding costs net of benefits |
| **MVA** | Subtracted (reduces value) | Cost of funding initial margin |
| **KVA** | Subtracted (reduces value) | Cost of regulatory capital |

Hull (OFD Chapter 9) notes a fundamental divide: "Financial economists have no problem with CVA and DVA, but have reservations about FVA, MVA, and KVA." We will explore why this debate persists and why practitioners often disagree with academics.

### 34.1.3 The Three Perspectives on Value

Understanding XVA requires distinguishing three different notions of "value":

| Perspective | What It Includes | Who Uses It |
|-------------|------------------|-------------|
| **Mid-Market (No-Default)** | $f_{\text{nd}}$ only | Quant models, academic pricing |
| **Accounting Fair Value** | $f_{\text{nd}} - \text{CVA} + \text{DVA}$ | Financial reporting (ASC 820, IFRS 13) |
| **Desk Economics** | $f_{\text{nd}} - \text{CVA} + \text{DVA} - \text{FVA} - \text{MVA} - \text{KVA}$ | P&L, trader compensation, deal approval |

> **Desk Reality: Why Sales and Trading Fight About Value**
>
> Sales: "The mid-market NPV is $2 million. Let's quote that to the client."
>
> Trader: "After XVA charges, I only net $1.4 million. If you quote mid, I lose money."
>
> Sales: "But CVA and FVA aren't 'real'—they're just accounting."
>
> Trader: "Tell that to my P&L. I get charged at inception and carry those reserves on my book."
>
> This tension is structural: Sales is measured on flow and client relationships; Trading is measured on risk-adjusted returns including XVA costs. The solution is transparency: agree on XVA methodology and show the client a "clean" price alongside the "all-in" price.

---

## 34.2 CVA: Credit Value Adjustment

### 34.2.1 Economic Intuition

The credit value adjustment (CVA) is the present value of the expected cost to the bank if its counterparty defaults. Hull (RM Chapter 20) defines it precisely: "The credit valuation adjustment (CVA) is the bank's estimate of the present value of the expected cost to the bank of a counterparty default."

When a counterparty defaults, several things happen in sequence:

1. An **event of default** triggers early termination of all outstanding derivatives
2. The portfolio's market value is calculated (the **settlement amount**)
3. If the portfolio has positive value to the bank, the bank becomes an **unsecured creditor**
4. The bank recovers only a fraction $R$ (the recovery rate) of what it is owed

CVA prices this potential loss. It is the present value of what the bank expects to lose, averaging over all possible default times and market scenarios.

> **Connection to Chapter 32:** The expected exposure $v_i$ in the CVA formula is exactly the quantity we developed in Chapter 32. The CVA calculation takes the exposure framework from Chapter 32 and prices it using the credit spreads and survival probabilities we'll develop fully in Chapters 36 and 41-42.

### 34.2.2 The CVA Formula

Hull (RM Chapter 20) provides the discrete-time formula. Suppose the life of the longest derivative is $T$ years, divided into $n$ intervals. Define:

- $q_i$: risk-neutral probability of default during interval $i$
- $v_i$: present value of expected exposure at the midpoint of interval $i$, conditional on default
- $R$: recovery rate

Then:

$$\boxed{\text{CVA} = \sum_{i=1}^{n} (1-R) \, q_i \, v_i}$$

This formula appears "deceptively simple," as Hull notes, but the implementation is "quite complicated and computationally very time-consuming." The challenge lies in computing the $v_i$ terms, which require simulating the entire portfolio forward in time.

**Intuition:** The formula says: for each possible default interval, multiply:
- Probability of defaulting in that interval ($q_i$)
- Loss given default ($1-R$)
- Expected exposure at that time ($v_i$)

Then sum across all intervals.

**Unit Check:**

| Quantity | Units | Interpretation |
|----------|-------|----------------|
| $(1-R)$ | dimensionless | Loss given default fraction |
| $q_i$ | dimensionless | Probability |
| $v_i$ | currency | PV of expected exposure |
| CVA | currency | Expected loss |

The product $(1-R) q_i v_i$ gives expected loss from defaults in interval $i$; summing over all intervals gives total CVA.

### 34.2.3 Computing Default Probabilities

The $q_i$ are **risk-neutral** default probabilities, estimated from the counterparty's credit spreads. Hull (RM Chapter 20) provides the link to hazard rates. If $s_i$ is the counterparty's credit spread for maturity $t_i$, the average hazard rate is:

$$\bar{\lambda}_i = \frac{s_i}{1-R}$$

The survival probability to time $t_i$ is $e^{-\bar{\lambda}_i t_i}$, so the probability of default during interval $i$ is:

$$\boxed{q_i = \exp\left(-\frac{s_{i-1} t_{i-1}}{1-R}\right) - \exp\left(-\frac{s_i t_i}{1-R}\right)}$$

This formula connects directly to the survival probability framework in Chapter 36 and the CDS hazard rate bootstrap in Chapter 42.

**Why risk-neutral probabilities?** Hull emphasizes that "the calculation of CVA involves the valuation of potential future cash flows and... it is correct to use risk-neutral, rather than real-world, default probabilities for valuation." The risk-neutral measure is appropriate because CVA is a price, not a risk metric.

> **Connection to CDS Market:** CVA uses the same survival probabilities as CDS pricing. If you can price single-name CDS (Chapter 41), you have the building blocks for CVA. A bank's CVA desk often calibrates counterparty default probabilities directly from observable CDS spreads using the techniques in Chapter 42.

### 34.2.4 Computing Expected Exposure

The $v_i$ require Monte Carlo simulation. Hull (RM Chapter 20) explains: "The market variables determining the future value of the transactions that the dealer has with the counterparty are simulated in a risk-neutral world between time zero and time $T$. On each simulation trial the exposure of the dealer to the counterparty at the midpoint of each interval is calculated."

The exposure at any time, after netting and collateral, is:

$$E(t) = \max(V(t) - C(t), 0)$$

where $V(t)$ is the portfolio value and $C(t)$ is collateral held. This connects to the exposure concepts developed in Chapter 32.

**The simulation process:**
1. Generate thousands of paths for all market variables (rates, FX, commodities, etc.)
2. At each time point $t_i^*$ (midpoint of interval $i$), compute portfolio value $V(t_i^*)$
3. Apply CSA rules to determine collateral $C(t_i^*)$
4. Compute exposure $E(t_i^*) = \max(V(t_i^*) - C(t_i^*), 0)$
5. Average exposures across paths to get expected exposure
6. Discount to get $v_i$

Hull notes that "dealers may have transactions with thousands of counterparties so that the calculation of the $v_i$ for all of them can be computationally very intensive."

### 34.2.5 Cure Period and Collateral Timing

A critical practical detail: collateral at default reflects the MTM from several days *earlier*. Hull (RM Chapter 20) explains: "The effect of the cure period is that the collateral at the time of a default does not reflect the value of the portfolio at the time of the default. It reflects the value 10 or 20 days earlier."

This creates a fundamental insight: **even perfectly collateralized trades carry residual exposure** equal to the potential MTM change during the cure period (also called the margin period of risk or MPOR).

**Example 34.1 — Cure Period Effect (from Hull RM Example 20.1)**

A bank has a zero-threshold collateral agreement with a counterparty. The cure period is 20 days. Consider what happens at default time $\tau$:

*Scenario A:* Portfolio value to bank at $\tau$ = 50. Value 20 days earlier = 45.
- Collateral at default = 45 (reflects old MTM)
- Exposure = $\max(50 - 45, 0) = 5$

*Scenario B:* Portfolio value to bank at $\tau$ = 50. Value 20 days earlier = 55.
- Collateral at default = 55 (more than current value)
- Exposure = $\max(50 - 55, 0) = 0$

*Scenario C:* Portfolio value to bank at $\tau$ = -50. Value 20 days earlier = -55.
- Bank has posted collateral of 55 with counterparty
- If counterparty defaults, only 50 needed but 55 posted
- Exposure = $\max(-50 - (-55), 0) = 5$ (excess collateral not returned)

The cure period creates exposure even under "perfect" zero-threshold collateralization. This is why post-crisis regulation focused heavily on reducing the MPOR through daily margining and standardized close-out procedures.

**Example 34.2 — Collateral Effect on CVA (Quantitative Illustration)**

To see how dramatically collateral reduces CVA, consider a 3-year portfolio with the following inputs:

| Interval | Midpoint | $\text{EE}$ (uncollat.) | $\text{EE}$ (collat.) | $D(0, t_i^*)$ | $\Delta\text{PD}_i$ |
|----------|----------|------------------------|----------------------|----------------|---------------------|
| 1 | 0.5 | \$10M | \$2M | 0.99 | 0.0198 |
| 2 | 1.5 | \$8M | \$1.5M | 0.97 | 0.0290 |
| 3 | 2.5 | \$6M | \$1M | 0.95 | 0.0373 |

Recovery $R = 40\%$ (LGD = 0.6). Default probabilities derived from piecewise-constant hazards $\lambda_1 = 2\%, \lambda_2 = 3\%, \lambda_3 = 4\%$ using the exponential survival recursion.

**Uncollateralized CVA:**
$$\text{CVA} = 0.6 \times (0.99 \times 10M \times 0.0198 + 0.97 \times 8M \times 0.0290 + 0.95 \times 6M \times 0.0373)$$
$$= 0.6 \times (196{,}020 + 225{,}040 + 212{,}610) = 0.6 \times 633{,}670 = \$380{,}202$$

**Collateralized CVA:**
$$\text{CVA}_c = 0.6 \times (0.99 \times 2M \times 0.0198 + 0.97 \times 1.5M \times 0.0290 + 0.95 \times 1M \times 0.0373)$$
$$= 0.6 \times (39{,}204 + 42{,}195 + 35{,}435) = 0.6 \times 116{,}834 = \$70{,}100$$

**Result:** Collateral reduces CVA by **81.6%** ($380K → $70K). The residual \$70K reflects cure-period exposure — even "perfect" collateralization leaves a gap.

**Sanity Check:** Better collateralization → lower CVA ✓. The residual amount reflects the margin period of risk, not a modeling error.

### 34.2.6 CVA After Accounting for Defaults

Taking CVA into account, the value of the derivatives portfolio to the bank becomes:

$$f_{\text{nd}} - \text{CVA}$$

This is a **unilateral adjustment**—it reflects only the counterparty's default risk, not the bank's own default risk. As we will see, the full picture requires DVA as well.

---

## 34.3 Worked Example: CVA for a Forward Contract

**Example 34.3 — Gold Forward CVA (from Hull RM Example 20.3)**

A bank enters a forward contract to buy 1 million ounces of gold from a mining company in 2 years at $1,500/oz. Current forward price is $1,600/oz.

**Given:**
- Default probability: 2% in year 1, 3% in year 2 (at midpoints)
- Recovery rate: $R = 30\%$
- Risk-free rate: $r = 5\%$
- Forward volatility: $\sigma = 20\%$

**Step 1: Calculate expected exposures**

For a forward contract, the exposure at time $t$ equals the option-like payoff $\max(F_t - K, 0)$ discounted. Hull (RM Chapter 20) shows that:

$$v_i = e^{-rT}[F_0 N(d_{1,i}) - K N(d_{2,i})]$$

where $d_{1,i} = \frac{\ln(F_0/K) + \sigma^2 t_i / 2}{\sigma \sqrt{t_i}}$ and $d_{2,i} = d_{1,i} - \sigma\sqrt{t_i}$

For $t_1 = 0.5$ years (midpoint of year 1):
- $d_{1,1} = \frac{\ln(1600/1500) + 0.04 \times 0.25}{0.2 \times 0.707} = 0.5271$
- $d_{2,1} = 0.3856$
- $N(d_{1,1}) = 0.701$, $N(d_{2,1}) = 0.650$
- $v_1 = e^{-0.10}[1600 \times 0.701 - 1500 \times 0.650] = \$132.38$ per ounce

For $t_2 = 1.5$ years (midpoint of year 2):
- $d_{1,2} = 0.3860$, $d_{2,2} = 0.1410$
- $N(d_{1,2}) = 0.650$, $N(d_{2,2}) = 0.556$
- $v_2 = e^{-0.10}[1600 \times 0.650 - 1500 \times 0.556] = \$186.65$ per ounce

**Step 2: Calculate CVA**

$$\text{CVA} = (1-0.30) \times (0.02 \times 132.38 + 0.03 \times 186.65)$$
$$= 0.70 \times (2.648 + 5.600) = 0.70 \times 8.248 = \$5.77 \text{ per ounce}$$

**Step 3: Adjust forward value**

No-default value: $(1600 - 1500)e^{-0.05 \times 2} = \$90.48$ per ounce

Value after CVA: $90.48 - 5.77 = \boxed{\$84.71}$ per ounce

For 1 million ounces: CVA = $5.77 million; adjusted value = $84.71 million.

**Sanity Check:** CVA is about 6.4% of the no-default value—reasonable for a 2-year trade with a 5% cumulative default probability and 70% loss given default.

---

## 34.4 DVA: Debit Value Adjustment

### 34.4.1 The Controversial "Gain from Default"

The debit (or debt) value adjustment (DVA) is the mirror image of CVA. Hull (RM Chapter 20) defines it: "DVA is the expected cost to the counterparty because the dealer might default. It is the counterparty's CVA. If DVA is a cost to the counterparty, it must be a benefit to the dealer."

The formula is symmetric to CVA:

$$\boxed{\text{DVA} = \sum_{i=1}^{n} (1-R^*) \, q_i^* \, v_i^*}$$

where:
- $q_i^*$: probability of default by the *bank* during interval $i$
- $v_i^*$: present value of the counterparty's exposure to the bank
- $R^*$: recovery rate if the bank defaults

### 34.4.2 Why DVA Exists: The Zero-Sum Argument

The logic for DVA rests on a fundamental principle: "Derivatives are zero-sum games." Hull (OFD Chapter 9) explains: "The gain to one side always equals the loss to the other side. If the bank's counterparty is worse off because of the possibility that the bank will default on the outstanding derivatives, the bank (or the bank's creditors) must be better off."

Without DVA, deals often cannot be agreed. Hull provides an illuminating example:

Consider two parties X and Y negotiating a swap with no-default fair rate of 2.2%.

- X calculates: "Accounting for Y's default risk, I should pay only 2.1%"
- Y calculates: "Accounting for X's default risk, I should receive 2.35%"

If each side considers only the *other's* default risk, there is no overlap—no deal gets done. DVA helps both sides value expected cashflows similarly, enabling trade.

### 34.4.3 Bilateral CVA

Taking both CVA and DVA into account, the value to the bank is:

$$\boxed{V = f_{\text{nd}} - \text{CVA} + \text{DVA}}$$

This is the **bilateral CVA** framework:
- CVA reduces value (cost of counterparty default)
- DVA increases value (benefit from potential own default)

The bilateral framework is now standard in accounting (ASC 820, IFRS 13) and is required for fair value reporting.

### 34.4.4 The Accounting Paradox and Phantom Equity

Hull (OFD Chapter 9) notes a "counterintuitive effect" of DVA: "As the bank's creditworthiness declines, DVA increases. This makes the derivatives portfolio more valuable to the bank."

When a bank's credit spreads widen:
- $q_i^*$ increases (higher default probability)
- DVA increases
- Reported profits increase

This seems perverse—the worse a bank does, the more profit it reports? The reason is that as default becomes more likely, the bank is "more likely that it will not have to honor its derivatives obligations."

> **Desk Reality: DVA as "Phantom Equity"**
>
> When your bank's credit spread widens, DVA "profit" appears because your liabilities are worth less in market terms. But this is **phantom equity**—you can only realize it by actually defaulting. Banks booked billions in DVA "profits" during 2008-2011 credit crises, but couldn't spend a dime of it.
>
> **The perverse incentive:** A struggling bank's traders might report record profits precisely when the bank is closest to failure. The worse the bank's credit, the higher the DVA gains.
>
> **The regulatory response:** Basel III requires banks to **deduct DVA gains from Tier 1 capital**. The accounting profit exists on the income statement but is not available to absorb losses. Regulators recognized that DVA gains don't represent real economic value that can protect depositors.
>
> **Middle office implication:** When reconciling accounting P&L to regulatory capital, you'll see a line item for "DVA add-back." This is the reversal of phantom profits that regulators won't count.

In 2011, Hull (RM Chapter 20) reports: "Some banks reported several billion dollars of profits from this source in the third quarter of 2011." The regulatory response was to exclude DVA gains and losses from the definition of common equity for regulatory capital purposes.

**Example 34.4 — DVA Gain from Spread Widening**

A bank has $500 million notional of swaps where it is out-of-the-money to counterparties (negative MTM to bank). The bank's 5-year CDS spread widens from 80bp to 180bp. Recovery assumption is 40%.

**Before:** 5-year survival probability = $e^{-0.008 \times 5 / 0.6} = 0.9355$
**After:** 5-year survival probability = $e^{-0.018 \times 5 / 0.6} = 0.8607$

Change in default probability = $0.9355 - 0.8607 = 0.0748$ (7.48%)

**Approximate DVA change:**
$$\Delta \text{DVA} \approx (1-R) \times \Delta(\text{default prob}) \times \text{Average negative exposure}$$

If average negative exposure (counterparty's claim) is $25 million:
$$\Delta \text{DVA} \approx 0.6 \times 0.0748 \times 25 = \$1.12 \text{ million profit}$$

**Interpretation:** The bank reports $1.12 million profit because its liabilities became less valuable. But this profit:
- Cannot be distributed as dividends
- Is not counted in regulatory capital
- Would only "realize" if the bank actually defaults

---

## 34.5 Interest Rate Swaps vs. Currency Swaps: Exposure Profiles

Hull (RM Chapter 20) provides an important comparison that illustrates why CVA differs dramatically across product types.

### 34.5.1 The Shape of Exposure Over Time

Consider a dealer with matched pairs of offsetting swaps—one with each of two counterparties. The **expected exposure profiles** differ strikingly between interest rate swaps and currency swaps:

**Interest Rate Swaps:** Expected exposure starts at zero, increases to a peak around the middle of the swap's life, then decreases back toward zero as maturity approaches. The "hump-shaped" profile arises because:
- At inception, the swap is at-market (zero value)
- As time passes, rate movements create potential value divergence
- Near maturity, very little remains to be exchanged, so exposure diminishes

**Currency Swaps:** Expected exposure increases steadily with the passage of time. Hull explains: "The main reason for the difference is that principals are exchanged at the end of the life of a currency swap and there is uncertainty about the exchange rate at that time. By contrast, toward the end of the life of the interest rate swap, very little is still to be exchanged."

### 34.5.2 Implications for CVA

The $q_i$ that the dealer calculates for counterparties are the same regardless of the transaction type—they depend only on the counterparty's credit. But the $v_i$ are on average much greater for currency swaps than for interest rate swaps of comparable maturity and notional.

This explains why:
- CVA charges on currency swaps are typically much higher than on interest rate swaps
- Cross-currency basis swaps carry significant counterparty risk
- The MPOR is particularly important for FX products (exchange rate can move substantially in 10-20 days)

---

## 34.6 FVA: Funding Value Adjustment

### 34.6.1 The Funding Problem

Consider a dealer who enters an interest rate swap with a corporate end user and hedges with another bank through a CCP:

```
End User ←→ Bank A ←→ Bank B (via CCP)
    (bilateral)     (cleared)
```

The swap with the end user is uncollateralized. The hedge with Bank B requires margin. Hull (OFD Chapter 9) explains the asymmetry: when the hedge has negative value to Bank A, it posts variation margin with the CCP but receives nothing from the end user. This creates a **funding need**.

The **funding valuation adjustment (FVA)** captures the cost of funding derivative positions when there is asymmetric collateralization.

### 34.6.2 FCA and FBA

Hull defines two components:
- **FCA** (Funding Cost Adjustment): present value of expected future funding costs
- **FBA** (Funding Benefit Adjustment): present value of expected future funding benefits

$$\boxed{\text{FVA} = \text{FCA} - \text{FBA}}$$

When an uncollateralized derivative has positive value, there is a funding cost (the bank has an asset it cannot finance via collateral). When it has negative value, there is a funding benefit (the bank receives an implicit loan from the counterparty).

### 34.6.3 The Theory vs. Practice Debate

Here is where financial economics and dealer practice diverge sharply.

**Practitioner view:** If the bank's average funding cost is Fed funds + 100bp, and interest on margin is Fed funds - 20bp, then the funding cost is 120bp per year. FVA should use this spread.

**Finance theory:** Hull (OFD Chapter 9) invokes the classic Modigliani-Miller argument: "The way a project is funded should not affect the required return on the investment. The required return should reflect the riskiness of the project."

Margin is low-risk. If we accept it earns only Fed funds + 10bp, then the funding cost is 30bp, not 120bp.

Hull provides a dialogue capturing the essence of the debate:

> **Financial Economist:** The cost you should use for funding a project should reflect the risk of the project.
>
> **Financial Engineer:** But tying up funds in initial margin prevents me from using the funds elsewhere. There is a cost to low-risk, low-return projects.
>
> **Financial Economist:** You talk as though funds are in short supply. If you have good projects, the market will provide funding.
>
> **Financial Engineer:** I am not sure that is how things work in practice.

Hull (OFD Chapter 9) observes: "This argument is over 50 years old in the finance literature and so its theoretical validity has stood the test of time. Many practitioners disagree with the theory."

### 34.6.4 The FVA Asymmetry in Practice

> **Desk Reality: The FVA Screaming Match**
>
> Sales: "Client wants to trade at mid-market."
>
> Trader: "Treasury charges me SOFR + 50bp to fund this! Who pays for that?"
>
> **The asymmetry that academics don't see:** Banks routinely charge clients for FCA (funding cost adjustment) but rarely credit FBA (funding benefit adjustment). If the bank can borrow at SOFR + 50bp to post collateral, and the trade requires $10M collateral for 5 years, that's $250K in funding costs someone must eat.
>
> When the trade goes the other way (bank receives implicit funding from an uncollateralized liability), does the bank pay the client? Almost never. The client doesn't know to ask, and the bank isn't volunteering.
>
> **Why this matters for pricing:** Clients dealing with multiple banks will see different all-in prices depending on each bank's funding cost and FVA policy. A bank with AA credit and low funding costs can offer better prices than a BBB bank—even if their mid-market models agree exactly.

### 34.6.5 Worked Example: FVA from Expected Funding Requirement

> **Implementation note (policy-dependent):** The sources define FCA/FBA conceptually but do not give a single canonical discrete-sum formula for FVA in terms of a desk-ready “funding requirement profile.” The right implementation depends on your funding policy (what constitutes “funding requirement,” whether FBA is recognized, what curve is used for discounting, and whether funding is computed at trade, netting-set, or bank level). The following mapping is **illustrative**:

$$\text{FCA} \approx \sum_{i=1}^{N} D(0, t_i^*) \cdot s_f \cdot F(t_i^*) \cdot \Delta t$$

where $s_f$ is the funding spread over the discounting rate (units: 1/year), $F(t)$ is the expected funding requirement (currency), and $\Delta t$ is the year fraction.

**Example 34.5 — FCA Calculation (Toy)**

| Interval | Midpoint | Funding Requirement $F$ | $D(0, t_i^*)$ |
|----------|----------|------------------------|----------------|
| 1 | 0.5 | \$10M | 0.99 |
| 2 | 1.5 | \$8M | 0.97 |
| 3 | 2.5 | \$6M | 0.95 |

Funding spread $s_f = 1\%$ per year, $\Delta t = 1$ year per interval.

$$\text{FCA} = 0.99 \times 0.01 \times 10M \times 1 + 0.97 \times 0.01 \times 8M \times 1 + 0.95 \times 0.01 \times 6M \times 1$$
$$= 99{,}000 + 77{,}600 + 57{,}000 = \boxed{\$233{,}600}$$

If the bank also recognizes funding benefit (FBA) for negative funding requirement, it computes an analogous term and nets: $\text{FVA} = \text{FCA} - \text{FBA}$.

### 34.6.6 MVA: Margin Value Adjustment

The **margin valuation adjustment (MVA)** specifically captures the cost of funding initial margin. As initial margin requirements have increased post-crisis (both for CCP-cleared and bilateral trades under SIMM), MVA has grown in importance.

Unlike FVA, MVA must be calculated on a portfolio basis for CCP transactions, because the CCP looks at all trades when determining margin requirements. Hull (OFD Chapter 9) notes that the incremental MVA "has to be calculated on a portfolio basis. In the case of... it is the impact of a new transaction on the initial margin required by the CCP for portfolio that Bank A is clearing through the CCP that determines the incremental initial margin requirements and therefore MVA."

---

## 34.7 KVA: Capital Value Adjustment

### 34.7.1 The Capital Cost Problem

Regulatory capital requirements tie up equity. If shareholders require 15% returns and a trade requires $10 million of additional capital, should the trade earn 15% on that capital?

**KVA** (capital valuation adjustment) is a charge to reflect incremental capital costs.

**Practitioner view:** We need to earn the hurdle rate on incremental capital. KVA ensures we do.

**Finance theory:** As the bank uses more equity, it becomes less risky, and the required return on equity should fall. The marginal cost of capital for a low-risk project is lower than the average.

Hull (OFD Chapter 9) provides another dialogue:

> **Financial Economist:** You do not need to make a KVA. As long as a derivatives book provides a return reflecting its risk your investors will be happy.
>
> **Practitioner:** Equity capital requirements for derivatives have gone up since the crisis. My equity investors require a 15% per annum return. If I enter into a derivatives transaction that requires additional capital under the new regulations, I need to make sure that the return on that capital is at least 15%.
>
> **Financial Economist:** But as more of the bank is financed by equity capital it becomes less risky and the return required by equity investors goes down... So the marginal return required on new equity capital is low.
>
> **Practitioner:** I am not sure that is how things work in practice.

### 34.7.2 KVA and Basel III Capital Requirements

> **Desk Reality: The Balance Sheet is Not Free**
>
> Every derivative consumes regulatory capital under Basel III. The calculation involves:
>
> 1. **Exposure at Default (EAD):** A supervisory measure of counterparty exposure based on replacement cost (today's MTM after collateral) plus an add-on meant to capture potential future exposure (PFE). The exact formula depends on the regulatory approach (e.g., SA-CCR vs internal models) and your jurisdiction/bank implementation.
>
> 2. **Risk-Weighted Assets (RWA):** Apply a counterparty risk weight to EAD (how the risk weight is set depends on the approach: standardized vs IRB, rating vs internal PD/LGD).
>
> 3. **Required Capital:** Multiply RWA by the capital ratio required by regulation + buffers (your bank typically uses an internal “all-in” ratio).
>
> 4. **KVA:** Convert required capital into a pricing charge by applying a cost of capital over the horizon (in practice often via a discounted sum/integral across future capital profiles).
>
> **Toy example (illustrative numbers):** A 5-year $100M interest rate swap with a BBB corporate:
> - Assume EAD (per your capital model) = $5M
> - Assume risk weight = 100%
> - Assume all-in capital ratio = 12% → required capital = $600K
> - If cost of capital = 15%, annual capital cost ≈ $90K
> - Over 5 years (discounted) → on the order of a few hundred thousand dollars
>
> This is why desks care about capital consumption: even if mid-market NPV is near zero, the “all-in” economics after KVA can change whether the trade clears the hurdle.

### 34.7.3 KVA Formula

The general KVA formula integrates capital requirements over the life of the trade:

$$\boxed{\text{KVA} = \int_0^T h_c \times K(t) \times d(t) \, dt}$$

where:
- $h_c$ = hurdle rate (cost of capital minus risk-free rate)
- $K(t)$ = expected regulatory capital at time $t$
- $d(t)$ = discount factor to time $t$

In discrete form:
$$\text{KVA} = \sum_{i=1}^{n} h_c \times K_i \times \Delta t_i \times d_i$$

### 34.7.4 The Complete XVA Stack

Putting it all together:

$$V_{\text{all-in}} = f_{\text{nd}} - \text{CVA} + \text{DVA} - (\text{FCA} - \text{FBA}) - \text{MVA} - \text{KVA}$$

All XVAs are "computationally time-consuming to calculate. Monte Carlo simulations are necessary to determine expected credit exposures, expected funding costs, and expected capital requirements at future times."

---

## 34.8 The Deal Ticket: Putting It All Together

**Example 34.6 — Comprehensive XVA Decomposition**

A derivatives desk is quoting a 7-year $50 million notional receiver swap (bank receives fixed, pays floating) to a BBB-rated corporate client. The swap will be uncollateralized.

**Given:**
- Mid-market NPV: +$1,200,000 (to bank)
- Counterparty CDS spread: 150bp (5-year), recovery = 40%
- Bank funding cost: SOFR + 45bp
- Expected peak exposure: $4.5M (occurs around year 4)
- Regulatory capital requirement: $2.2M (under SA-CCR)
- Bank's cost of capital: 12% (target ROE)
- Risk-free rate (for discounting KVA): 4%

**Step 1: CVA Calculation**

Using the simplified CVA formula with average expected exposure:

5-year survival probability: $e^{-0.015 \times 5/0.6} = 0.8825$

Approximate 7-year cumulative default probability ≈ 15%

Average expected exposure (discounted) ≈ $2.5M

$$\text{CVA} \approx (1-0.40) \times 0.15 \times \$2.5M = \$225,000$$

**Step 2: DVA Calculation**

Assume bank CDS spread: 60bp, bank's negative exposure (when swap is in-the-money to counterparty) averages $1.5M.

Bank's 7-year default probability ≈ 6%

$$\text{DVA} \approx (1-0.40) \times 0.06 \times \$1.5M = \$54,000$$

**Step 3: FVA Calculation**

The swap has positive value, so the bank has a funding cost. Average positive MTM over the life ≈ $800K.

Funding spread = 45bp

$$\text{FVA} \approx 0.0045 \times \$800K \times 5 \text{ (effective duration)} = \$18,000$$

**Step 4: MVA Calculation**

Bilateral IM under SIMM ≈ $1.8M

Funding cost of IM = 45bp

$$\text{MVA} \approx 0.0045 \times \$1.8M \times 6 \text{ (years)} = \$48,600$$

**Step 5: KVA Calculation**

Capital requirement: $2.2M

Cost of capital above risk-free: 12% - 4% = 8%

$$\text{KVA} \approx 0.08 \times \$2.2M \times 5 \text{ (effective duration)} = \$880,000$$

**Step 6: Deal Ticket Summary**

| Component | Amount | Sign |
|-----------|--------|------|
| Mid-Market NPV | $1,200,000 | + |
| CVA | $(225,000) | − |
| DVA | $54,000 | + |
| FVA | $(18,000) | − |
| MVA | $(48,600) | − |
| KVA | $(880,000) | − |
| **Net Desk Economics** | **$82,400** | |

**Interpretation:** The "profitable" $1.2M trade actually nets only $82K for the desk after XVA charges. The KVA alone consumes $880K—over 70% of the gross NPV.

> **Desk Reality: The Deal/No-Deal Decision**
>
> At $82K net, should the desk do this trade?
>
> **Arguments for:**
> - Client relationship value
> - Cross-sell opportunity
> - Flow that improves hedging efficiency elsewhere
>
> **Arguments against:**
> - ROE on $2.2M capital = $82K / $2.2M / 7 years = 0.5% per year
> - Bank's hurdle rate is 12%—this trade destroys value
> - Capital better deployed elsewhere
>
> **The XVA desk's role:** Provide these numbers so the decision is informed. Don't let sales book a trade as "$1.2M profit" when desk economics say $82K.

---

## 34.9 Wrong-Way Risk

### 34.9.1 Definition

Hull (RM Chapter 20) defines the key concept: "A situation where there is a positive dependence between the probability of default and the exposure is referred to as **wrong-way risk**. A situation where there is negative dependence is referred to as **right-way risk**."

- **Wrong-way risk:** Default more likely when exposure is high → CVA understated
- **Right-way risk:** Default more likely when exposure is low → CVA overstated

The standard CVA formula assumes independence between $q_i$ and $v_i$. Wrong-way risk means this assumption is violated in a dangerous direction.

### 34.9.2 When Wrong-Way Risk Arises

**Example: CDS Protection Selling**

A counterparty sells credit protection via CDS. When the reference entity's spreads widen:
1. The CDS has positive value to the dealer (exposure increases)
2. The protection seller's credit likely worsens (correlated spreads)
3. Default probability increases exactly when exposure is highest

This is wrong-way risk. AIG is the canonical example—it sold massive amounts of credit protection, and when credit markets deteriorated, both its exposure and its default probability spiked simultaneously.

**Example: Speculating Counterparty**

Hull (RM Chapter 20) notes: "A situation in which a company is speculating by entering into many similar trades with one or more dealers is likely to lead to wrong-way risk for these dealers. This is because the company's financial position and therefore its probability of default is likely to be affected adversely if the trades move against the company."

> **Desk Reality: When Exposure and Default Align**
>
> Consider a receiver swap with a struggling corporate: you receive fixed, they pay floating. If rates fall, your exposure rises (swap is in-the-money to you). But falling rates often coincide with economic stress—exactly when corporates default. Your exposure is highest when they're most likely to default.
>
> **How XVA desks handle it:** Apply **wrong-way risk multipliers** (sometimes 1.5x-2x CVA) for exposures that correlate with counterparty credit quality. Sophisticated desks model the correlation explicitly.

### 34.9.3 Right-Way Risk

When a counterparty is hedging an existing exposure, right-way risk can occur. Hull explains: "If a company enters into transactions with a dealer to partially hedge an existing exposure, there should in theory be right-way risk. This is because, when the transactions move against the counterparty, it will be benefiting from the unhedged portion of its exposure so that its probability of default should be relatively low."

### 34.9.4 Quantifying Wrong-Way Risk

The CVA formula assumes independence between $q_i$ and $v_i$. Wrong-way risk means this assumption is violated.

Basel II rules use an "alpha" multiplier to address this:
- Minimum alpha = 1.2 (CVA must be at least 20% higher than independence-based model)
- Default alpha = 1.4 (if bank has no own model)

Hull (RM Chapter 20) reports: "Estimates of alpha reported by banks range from 1.07 to 1.10."

More sophisticated approaches model hazard rates as functions of observable market variables. Hull and White (2012) propose a model where "the hazard rate at time $t$ is a function of variables observable at that time." The parameter describing the dependence can be estimated by relating past credit spreads for the counterparty to what the portfolio value would have been historically.

### 34.9.5 Worked Example: Wrong-Way Risk Impact on CVA

**Example 34.7 — Independence vs. Wrong-Way Risk (Two-State Model)**

Consider a single-interval setup with LGD = 0.6 and two equally likely states:

| State | Probability | Exposure $E$ | Default Prob $p$ |
|-------|------------|--------------|------------------|
| A ("bad") | 50% | \$20M | 5% |
| B ("good") | 50% | \$2M | 1% |

**Independence-based CVA (using averages):**
- Average exposure: $\bar{E} = 0.5 \times 20 + 0.5 \times 2 = \$11M$
- Average PD: $\bar{p} = 0.5 \times 0.05 + 0.5 \times 0.01 = 3\%$

$$\text{CVA}_{\text{ind}} = 0.6 \times 11M \times 0.03 = \$198{,}000$$

**Wrong-way risk CVA (expected product, acknowledging correlation):**

$$\mathbb{E}[E \cdot p] = 0.5 \times (20M \times 0.05) + 0.5 \times (2M \times 0.01) = 0.5 \times 1{,}000{,}000 + 0.5 \times 20{,}000 = 510{,}000$$

$$\text{CVA}_{\text{WWR}} = 0.6 \times 510{,}000 = \boxed{\$306{,}000}$$

**Increase due to wrong-way risk:** $306{,}000 - 198{,}000 = \$108{,}000$ (+54.5%)

**Takeaway:** The positive correlation between exposure and default probability materially increases CVA. The independence assumption — which the standard CVA formula relies on — understates CVA by over 50% in this stylized example. This is why Basel imposes the alpha multiplier (minimum 1.2) and why sophisticated desks model the correlation explicitly.

---

## 34.10 The XVA Desk: Organization and Transfer Pricing

### 34.10.1 How Banks Structure XVA Management

Banks organize XVA management in one of two ways:

**Centralized XVA Desk:**
- A dedicated desk calculates and owns all XVA P&L
- Trading desks are "charged" XVA at trade inception
- XVA desk hedges the aggregate CVA/DVA exposure
- Most large banks use this model

**Embedded in Business Lines:**
- Each trading desk owns its own XVA
- Desks hedge their own CVA exposure
- Coordination is informal
- More common at smaller institutions

> **Desk Reality: Who Owns the P&L?**
>
> **At trade inception:**
> - Trader shows +$1.2M NPV (mid-market)
> - XVA desk charges $400K (CVA + FVA + KVA, net of DVA)
> - Trader's day-one P&L: $800K
> - XVA desk books a $400K reserve
>
> **Over the trade's life:**
> - Trader's P&L: changes in mid-market value
> - XVA desk P&L: changes in XVA values (spread moves, exposure changes)
>
> **At maturity/close-out:**
> - XVA reserve releases (if no default occurred)
> - XVA desk shows profit equal to original charge minus hedging costs
>
> **The conflict:** Traders want low XVA charges; XVA desk wants conservative reserves. Both report to the same P&L, so there's tension about "fair" charging.

### 34.10.2 XVA Reserve vs. XVA Charge

A common source of confusion:

| Concept | Timing | Nature |
|---------|--------|--------|
| **XVA Charge** | At trade inception | Upfront cost to the trading desk |
| **XVA Reserve** | Over trade life | Liability on the balance sheet, released as risk declines |

The charge is what hits the trader's P&L immediately. The reserve is what the bank carries until the exposure resolves (trade matures, counterparty defaults, or trade is closed out).

### 34.10.3 Regulatory vs. Accounting vs. Economic XVA

| Type | Purpose | What's Included |
|------|---------|-----------------|
| **Accounting XVA** | Financial reporting (GAAP/IFRS) | CVA + DVA (required); FVA (optional but common) |
| **Regulatory XVA** | Capital calculations | CVA capital charge (Basel III); DVA excluded from capital |
| **Economic XVA** | Deal approval, pricing | All: CVA, DVA, FVA, MVA, KVA |

Banks must reconcile these three versions. The DVA treatment is particularly complex: it appears in accounting income but is excluded from regulatory capital.

---

## 34.11 Collateralized vs. Uncollateralized Economics

### 34.11.1 The Collateral Trade-Off

> **Desk Reality: The Collateral Trade-Off**
>
> With full daily collateralization (zero threshold, zero MTA):
> - **CVA ≈ 0** (exposure limited to MPOR, ~10 days)
> - **DVA ≈ 0** (symmetric)
> - **FVA ≈ 0** (collateral received funds collateral posted)
> - **But MVA appears**: cost of posting initial margin
>
> The economics shift, but don't disappear. Clearing mandates pushed CVA/FVA into MVA. The XVA desk's job got more complex, not simpler.

### 34.11.2 Cleared vs. Bilateral Comparison

| Aspect | Bilateral Uncollateralized | Bilateral with CSA | CCP Cleared |
|--------|---------------------------|-------------------|-------------|
| CVA | High | Low (MPOR residual) | Near zero |
| FVA | Can be large | Moderate | Near zero |
| MVA | Zero | Moderate (if IM required) | High |
| Margin calls | None | Daily/weekly | Daily |
| Close-out risk | High | Moderate | Low |
| Documentation | ISDA Master | ISDA + CSA | CCP rules |

---

## 34.12 CVA Risk Management

### 34.12.1 CVA as a Derivative

Hull (RM Chapter 20) makes a key observation: "A dealer has one CVA for each counterparty. These CVAs can themselves be regarded as derivatives. They are particularly complex derivatives."

When CVA increases, reported income decreases. Dealers hedge CVAs like other derivatives, which requires computing Greek letters for CVA with respect to:
1. **Market variables** affecting $v_i$ (rates, FX, commodities)
2. **Credit spreads** affecting $q_i$

### 34.12.2 CVA Sensitivity to Credit Spreads

From the CVA formula, the sensitivity to a parallel shift $\Delta s$ in credit spreads is (Hull RM equation 20.5):

$$\Delta(\text{CVA}) \approx \sum_{i=1}^{n}\left[t_i e^{-s_i t_i/(1-R)} - t_{i-1} e^{-s_{i-1} t_{i-1}/(1-R)}\right] v_i \, \Delta s + \text{(gamma terms)}$$

Basel III requires market risk capital for CVA risk arising from credit spread changes. However, Hull notes that "risks arising from changes in the market variables affecting the $v_i$ are not included in market risk capital calculations. This is presumably because they are more difficult to calculate."

This asymmetry has drawn criticism: "Sophisticated dealers who are capable of quantifying the $v_i$ risks have complained that, if they hedge these risks, they will be increasing their capital requirements. This is because the hedging trades would be taken into account in determining market risk capital whereas the CVA exposure to the market variables would not."

**Example 34.8 — CVA01: Sensitivity to a 1bp Credit Spread Bump**

Using the spread–hazard approximation $\lambda \approx s/(1-R)$, a 1bp increase in the counterparty's credit spread ($\Delta s = 0.0001$) translates to a hazard bump of:

$$\Delta\lambda = \frac{0.0001}{1-R} = \frac{0.0001}{0.6} = 0.0001667 \text{ per year}$$

Consider a base CVA of \$380,202 (from Example 34.2, with piecewise hazards $\lambda = [0.02, 0.03, 0.04]$, using the uncollateralized exposure profile). Bumping each hazard by 0.0001667 and recomputing the survival probabilities and default probability increments yields slightly higher $q_i$ values, giving:

$$\text{CVA}_{+1\text{bp}} \approx \$382{,}225$$

Therefore:

$$\boxed{\text{CVA01} = \text{CVA}_{+1\text{bp}} - \text{CVA}_{\text{base}} \approx \$2{,}023 \text{ per basis point}}$$

**Interpretation:** A 10bp widening in the counterparty's credit spread would increase CVA by approximately \$20,000. This sensitivity is what the CVA desk hedges using single-name CDS.

### 34.12.3 CVA Hedging

Dealers can hedge CVA by:
1. **Buying CDS protection** on counterparties
2. **Hedging market exposures** (rates, FX deltas)
3. **Adjusting deal terms** to reduce exposure

Hull (RM Chapter 20) notes that dealers "sometimes buy protection against their counterparties defaulting using credit default swaps or similar instruments."

---

## 34.13 Worked Example: Simple CVA for a Swap

**Example 34.9 — IRS CVA Calculation**

A bank has a 5-year pay-fixed swap with a counterparty. The swap has:
- Notional: $100 million
- Current MTM to bank: $2 million (positive)
- Expected future exposure profile (annual midpoints, after collateral):

| Year | $t_i^*$ | $EE_i$ (\$ millions) |
|------|---------|----------------------|
| 1 | 0.5 | 1.5 |
| 2 | 1.5 | 2.5 |
| 3 | 2.5 | 2.8 |
| 4 | 3.5 | 2.2 |
| 5 | 4.5 | 1.5 |

The counterparty has 5-year credit spread of 200bp (flat term structure). Recovery rate = 40%.

**Step 1: Calculate annual default probabilities**

Using the approximation $q_i \approx \Delta Q_i = Q(t_{i-1}) - Q(t_i)$ where survival probability $Q(t) = e^{-st/(1-R)}$:

With $s = 0.02$ and $1-R = 0.60$:
- $Q(1) = e^{-0.02 \times 1/0.60} = e^{-0.0333} = 0.9672$
- $Q(2) = e^{-0.0667} = 0.9355$
- $Q(3) = e^{-0.10} = 0.9048$
- $Q(4) = e^{-0.1333} = 0.8752$
- $Q(5) = e^{-0.1667} = 0.8465$

Annual default probabilities:
- $q_1 = 1 - 0.9672 = 0.0328$
- $q_2 = 0.9672 - 0.9355 = 0.0317$
- $q_3 = 0.9355 - 0.9048 = 0.0307$
- $q_4 = 0.9048 - 0.8752 = 0.0296$
- $q_5 = 0.8752 - 0.8465 = 0.0287$

**Step 2: Discount expected exposures**

Using OIS discount factors (approximated):
- $P(0.5) = 0.98$, $P(1.5) = 0.94$, $P(2.5) = 0.90$, $P(3.5) = 0.87$, $P(4.5) = 0.84$

PV of exposures:
- $v_1 = 1.5 \times 0.98 = 1.47$ million
- $v_2 = 2.5 \times 0.94 = 2.35$ million
- $v_3 = 2.8 \times 0.90 = 2.52$ million
- $v_4 = 2.2 \times 0.87 = 1.91$ million
- $v_5 = 1.5 \times 0.84 = 1.26$ million

**Step 3: Compute CVA**

$$\text{CVA} = (1-0.40) \sum_{i=1}^{5} q_i v_i$$
$$= 0.60 \times (0.0328 \times 1.47 + 0.0317 \times 2.35 + 0.0307 \times 2.52 + 0.0296 \times 1.91 + 0.0287 \times 1.26)$$
$$= 0.60 \times (0.0482 + 0.0745 + 0.0774 + 0.0566 + 0.0362)$$
$$= 0.60 \times 0.2929 = \boxed{\$175,700}$$

**Interpretation:** The 5-year swap has CVA of about $176,000, or roughly 8.8 basis points per annum running cost on the $100 million notional.

---

## 34.14 Practical Implementation Notes

### 34.14.1 What Goes Into an XVA System

A production XVA system implements the following end-to-end workflow:

**Step 1: Define Legal/Portfolio Scope**
- Identify netting sets (ISDA master agreement scope) and portfolio membership
- Capture early termination logic and settlement conventions
- CSA terms: thresholds, minimum transfer amount, independent amount / initial margin, eligible collateral, haircuts, rehypothecation status

**Step 2: Define Collateral Mechanics**
- Margin frequency (daily vs less frequent), collateral currencies, interest on cash collateral
- Cure period / MPOR (often 10–20 days)
- Exposure at default uses $E = \max(V - C, 0)$

**Step 3: Generate Future Market Scenarios**
- Choose models for relevant risk factors (rates, FX, equities, commodities, volatilities)
- Simulate under risk-neutral dynamics for CVA valuation
- Choose time grid $\{t_i^*\}$ aligned with product features (exercise dates, resets, payment dates)

**Step 4: Revalue Portfolio in Each Scenario**
- For each path and each midpoint $t_i^*$, compute portfolio MTM $V(t_i^*)$
- For cure-period modeling, also compute $V(t_i^* - c)$ to infer collateral at $t_i^*$

**Step 5: Compute Exposures**
- Pathwise net exposure: $E(t_i^*) = \max(V(t_i^*) - C(t_i^*), 0)$
- Expected exposure $\text{EE}(t_i^*)$: average across Monte Carlo trials
- Peak exposure / PFE: high percentile at each midpoint (e.g., 97.5%)

**Step 6: Compute Default Probabilities**
- Build counterparty risk-neutral default curve from credit spreads / CDS / bond-implied
- Use spread-to-hazard approximation $\bar{\lambda} = s/(1-R)$ or bootstrap full CDS curve (Chapter 42)
- Convert to interval probabilities $q_i = Q(t_{i-1}) - Q(t_i)$

**Step 7: Assemble CVA and DVA**
- CVA: combine EE, LGD, discounting, and $q_i$ via discrete sum
- DVA: same mechanics with own default probabilities and negative exposure

**Step 8: Compute FVA (Policy-Dependent)**
- Identify funding requirements: uncollateralized positive MTM, posted margin
- Choose funding spread and whether FBA is recognized
- Note: FVA implementation is policy-driven (see Section 34.6)

**Step 9: Produce Sensitivities and P&L Explain**
- CVA spread sensitivity (CVA01): relationship between $q_i$ and credit spreads
- Exposure Greeks: sensitivities of $v_i$ to market variables
- Incremental XVA: stored simulation paths for new-trade pricing (see Section 34.14.2)

Hull notes that "dealers may have transactions with thousands of counterparties so that the calculation of the $v_i$ for all of them can be computationally very intensive."

### 34.14.2 Incremental XVA for New Trades

When a new trade is proposed, the desk needs to know its incremental CVA impact. Recalculating the full Monte Carlo is impractical.

Hull (RM Chapter 20) describes the solution: "When the CVA calculations... are carried out, the paths followed by market variables on each simulation trial and the value of the portfolio on each simulation trial are stored. When a potential new transaction is being considered, its value at the future times is calculated for the values of the market variables that were obtained on the simulation trials."

The stored-path approach allows rapid "incremental XVA" pricing for trade approval by computing:
1. New trade's value on each stored path at each time
2. Incremental impact on portfolio value and exposure
3. Incremental effect on CVA via $\sum (1-R) q_i \Delta v_i$

### 34.14.3 Machine Learning Applications

Given the computational intensity, some banks use neural networks to approximate XVA calculations. Hull (OFD Chapter 9) notes: "Once the network has been constructed, calculating the target from the features is very fast."

The approach:
1. Generate many random portfolio configurations
2. Compute XVAs via full Monte Carlo
3. Train neural network to replicate results
4. Use network for real-time incremental XVA

This allows rapid responses to trade inquiries while maintaining accuracy calibrated to full Monte Carlo.

### 34.14.4 Validation and Sanity Checks

**Core Monotonicity Checks:**
CVA should increase when expected exposure increases, default probabilities/hazards increase, or recovery decreases (LGD increases). All follow directly from the discrete-sum structure. DVA should increase when expected negative exposure increases and own hazard increases.

**Collateral Sanity Check:**
Under idealized daily two-way VM with zero threshold and no cure-period lag, exposure should be near zero → CVA near zero. Residual exposure arises precisely from the cure period and collateral mismatch (see Example 34.2).

**Peak Exposure (PFE) Sanity:**
PFE (high percentile) should exceed average exposure at each time; maximum PFE should occur near periods of high optionality or volatility.

**P&L Explain (Daily XVA Decomposition):**
CVA P&L decomposes into:
- Credit-spread moves ($q_i$ moves)
- Exposure/market-factor moves ($v_i$ moves)

DVA P&L similarly depends on own spread and expected negative exposure. Any unexplained residual should be investigated — common causes include time discretization error (coarse grids missing exposure peaks), inconsistent CSA modeling, or mixed discounting frameworks.

**Implementation Pitfalls:**

| Pitfall | Consequence | Fix |
|---------|-------------|-----|
| Inconsistent netting/collateral modeling | Overstated or understated CVA | Ensure exposure computed on enforceable netting sets |
| Missing cure-period modeling | Understated exposure spikes | Value at $t_i^* - c$ to infer collateral at $t_i^*$ |
| Wrong sign conventions | P&L explain errors | Remember: $f_{\text{nd}} - \text{CVA} + \text{DVA}$ |
| Coarse time grids | Missed exposure peaks | Align grid with cashflow dates, resets, exercise dates |
| Mixing discounting with FVA | Double-counted funding | Document discounting + FVA policy together |
| Ignoring wrong-way risk | Understated CVA | Apply alpha multiplier or model correlation |

---

## 34.15 Summary

1. **XVAs are adjustments** to the no-default derivative value that account for real-world frictions: credit risk, funding costs, margin, and capital
2. **CVA** = present value of expected loss from counterparty default; reduces derivative value
3. **DVA** = present value of expected gain from own default; increases value but is "phantom equity"—excluded from regulatory capital
4. **Bilateral CVA** = $f_{nd} - \text{CVA} + \text{DVA}$
5. **FVA, MVA, KVA** address funding, margin, and capital costs; theory vs practice debate on whether they should exist, but practitioners charge them
6. **KVA** reflects the cost of regulatory capital; on capital-intensive trades it can be economically material
7. **Wrong-way risk** occurs when exposure and default probability are positively correlated; must be modeled or adjusted
8. **The XVA desk** centralizes these calculations and charges trading desks at inception
9. **Implementation** requires Monte Carlo simulation of all market variables and trades; machine learning helps speed real-time pricing
10. **Exposure profiles differ** dramatically by product type—currency swaps have much higher CVA than interest rate swaps of comparable notional

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| CVA | PV of expected loss from counterparty default | Cost of credit risk in derivatives |
| DVA | PV of expected gain from own default | Enables bilateral pricing agreement; "phantom equity" |
| Unilateral CVA | $f_{nd} - \text{CVA}$ | One-sided credit adjustment |
| Bilateral CVA | $f_{nd} - \text{CVA} + \text{DVA}$ | Two-sided credit adjustment |
| FVA | Adjustment for funding costs/benefits | Captures asymmetric collateralization |
| MVA | Cost of funding initial margin | Growing with margin requirements |
| KVA | Cost of regulatory capital | Links capital to pricing; can be economically material |
| Wrong-way risk | Positive correlation of exposure and PD | CVA understated if ignored |
| Cure period | Time between last margin call and close-out | Creates residual exposure |
| $v_i$ | PV of expected exposure at time $t_i$ | Key input to CVA formula |
| $q_i$ | Risk-neutral default probability in interval $i$ | From counterparty credit spreads |
| Alpha multiplier | Wrong-way risk adjustment (1.2-1.4) | Increases CVA to account for correlation |
| XVA desk | Centralized function managing all XVA | Charges desks, hedges aggregate exposure |

---

## Notation for This Chapter

| Symbol | Definition |
|--------|------------|
| $f_{\text{nd}}$ | No-default value of derivatives portfolio |
| CVA | Credit Value Adjustment |
| DVA | Debit (Debt) Value Adjustment |
| FVA | Funding Value Adjustment |
| FCA | Funding Cost Adjustment |
| FBA | Funding Benefit Adjustment |
| MVA | Margin Value Adjustment |
| KVA | Capital Valuation Adjustment |
| $q_i$ | Risk-neutral probability of counterparty default in interval $i$ |
| $v_i$ | PV of expected exposure at midpoint of interval $i$ |
| $R$ | Recovery rate (fraction recovered in default) |
| $s$ | Credit spread |
| $\bar{\lambda}$ | Average hazard rate (default intensity) |
| $E(t)$ | Exposure at time $t$ |
| $C(t)$ | Collateral at time $t$ |
| $c$ | Cure period / MPOR |
| $h_c$ | Hurdle rate (cost of capital minus risk-free rate) |
| $K(t)$ | Regulatory capital at time $t$ |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What does CVA measure? | The present value of expected loss to a bank from counterparty default on derivatives. |
| 2 | What does DVA measure? | The present value of expected gain to a bank from its own potential default (counterparty's CVA). |
| 3 | Write the bilateral CVA valuation formula. | $V = f_{\text{nd}} - \text{CVA} + \text{DVA}$ |
| 4 | Write the discrete CVA formula. | $\text{CVA} = \sum_{i=1}^{n} (1-R) q_i v_i$ |
| 5 | What are $q_i$ in the CVA formula? | Risk-neutral probability of counterparty default during interval $i$. |
| 6 | What is $v_i$ in the CVA formula? | Present value of expected net exposure at midpoint of interval $i$, conditional on default. |
| 7 | How is exposure calculated after netting and collateral? | $E = \max(V - C, 0)$ where $V$ is portfolio value and $C$ is collateral. |
| 8 | What is wrong-way risk? | Positive correlation between exposure and counterparty default probability. |
| 9 | What is right-way risk? | Negative correlation between exposure and default probability. |
| 10 | What is the cure period (MPOR)? | Time between last effective collateral call and close-out; typically 10-20 days. |
| 11 | Why does the cure period create exposure even with zero threshold? | Collateral at default reflects old MTM; exposure equals MTM change during cure period. |
| 12 | What is FVA? | Funding Value Adjustment: cost or benefit of funding derivative positions. |
| 13 | What is MVA? | Margin Value Adjustment: cost of funding initial margin requirements. |
| 14 | What is KVA? | Capital Value Adjustment: cost of regulatory capital tied up by transaction. |
| 15 | What is the theoretical controversy about FVA/MVA/KVA? | Finance theory says funding cost should reflect project risk, not average bank funding cost. |
| 16 | How does worsening bank credit affect DVA? | DVA increases as default probability rises, increasing reported portfolio value. |
| 17 | Why did regulators exclude DVA from capital? | To prevent banks from profiting in regulatory capital when their credit deteriorates. |
| 18 | How can CVA be hedged? | By buying CDS protection on counterparties; by hedging market exposure deltas. |
| 19 | What is the alpha multiplier for wrong-way risk? | Basel multiplier (min 1.2, default 1.4) to increase CVA for unmodeled wrong-way risk. |
| 20 | Can CVA be calculated trade-by-trade? | No, due to netting; it must be calculated at the netting set (counterparty) level. |
| 21 | Why do currency swaps have higher CVA than interest rate swaps? | Principal exchange at maturity creates exposure that grows with time; IRS exposure peaks mid-life. |
| 22 | How do dealers compute incremental CVA for new trades? | By storing Monte Carlo paths and revaluing only the new trade on those stored paths. |
| 23 | What is the "manufacturing cost" interpretation of XVA? | XVA charges are costs the desk pays: CVA (insurance), FVA (funding), KVA (capital rent). |
| 24 | Why can't you spend DVA profits? | DVA gains only realize upon your own default—"phantom equity." |
| 25 | What's the FVA asymmetry in practice? | Banks charge clients FVA but rarely pay FBA (funding benefit). |
| 26 | How does Basel III treat DVA? | Deducted from Tier 1 capital—not available to absorb losses. |
| 27 | When does collateral eliminate CVA? | With daily margining, zero threshold, zero MTA (exposure limited to MPOR). |
| 28 | What replaces CVA in the cleared world? | MVA (margin valuation adjustment) for initial margin costs. |
| 29 | What is wrong-way risk in the XVA context? | Exposure increases when counterparty credit deteriorates. |
| 30 | What's the academic argument against FVA? | FVA violates law of one price—funding costs shouldn't affect fair value. |
| 31 | What's the practitioner response to FVA critics? | Treasury charges real funding costs; desk must price them or lose money. |
| 32 | What is the KVA formula? | $\text{KVA} = \int h_c \times K(t) \times d(t) \, dt$ where $h_c$ = cost of capital spread, $K(t)$ = regulatory capital. |
| 33 | What is $f_{\text{nd}}$? | No-default value of the derivatives portfolio, assuming neither side defaults. |
| 34 | What is exposure with netting today? | $\max\left(\sum_{i=1}^{N} V_i, 0\right)$ — all trades netted before taking the max. |
| 35 | What is exposure without netting today? | $\sum_{i=1}^{N} \max(V_i, 0)$ — each trade's exposure computed individually. |
| 36 | State the hazard/intensity definition and its units. | $P(\tau \in (t, t+h] \mid \tau > t, \mathcal{F}_t) = \lambda_t h + o(h)$; $\lambda_t$ has units of 1/year. |
| 37 | Define FCA and FBA. | FCA = PV of expected future funding cost; FBA = PV of expected future funding benefit; FVA = FCA - FBA. |
| 38 | Should exposure simulation use risk-neutral or real-world dynamics? | Risk-neutral for CVA valuation (pricing); real-world may be used for scenario/PFE analysis. |
| 39 | What does an "event of default" trigger under ISDA? | The non-defaulting party has the right to terminate all transactions; events include bankruptcy, failure to pay, failure to post collateral. |
| 40 | Why does netting affect collateral requirements? | Collateral is computed on net portfolio value, not individual trades, reducing posted amounts. |

---

## Mini Problem Set

### Questions 1-6 (Solution sketches provided)

**1)** Define CVA, DVA, and bilateral CVA. Explain the sign of each adjustment and its economic meaning.

**Sketch:** CVA = cost of counterparty default (negative adjustment). DVA = benefit from own default (positive adjustment). Bilateral = $f_{nd} - CVA + DVA$. CVA reduces value because it is a cost; DVA increases value because it is a benefit.

---

**2)** A bank has one swap outstanding with a counterparty. Expected exposure at year 1 is $3 million, at year 2 is $2 million. Default probability each year is 2%. Recovery = 40%. Compute CVA assuming exposures are already present values.

**Sketch:**
$\text{CVA} = (1-0.4)(0.02 \times 3 + 0.02 \times 2) = 0.6 \times 0.10 = \$60,000$

---

**3)** Explain why the cure period creates exposure even under zero-threshold collateralization.

**Sketch:** Collateral at default equals MTM from $c$ days earlier. If MTM rose during cure period, collateral is less than current value → positive exposure. If MTM fell, bank may have excess collateral at counterparty → positive exposure from unreturned collateral.

---

**4)** What is wrong-way risk? Give an example involving CDS.

**Sketch:** Wrong-way risk = positive correlation between exposure and counterparty default probability. Example: Counterparty sells CDS protection. When reference credit deteriorates, CDS has positive value to dealer (exposure high) AND counterparty credit likely worsens (correlation). AIG is the canonical case.

---

**5)** A counterparty has a flat 5-year credit spread of 150bp. Assuming 40% recovery, what is the 5-year survival probability?

**Sketch:**
$Q(5) = \exp\left(-\frac{0.015 \times 5}{1 - 0.40}\right) = \exp\left(-\frac{0.075}{0.60}\right) = \exp(-0.125) = 0.8825$

Survival probability ≈ 88.25%. Default probability ≈ 11.75%.

---

**6)** Explain the theoretical objection to using average bank funding costs in FVA calculations.

**Sketch:** Finance theory says discount rates should reflect project risk, not funding source. Margin/collateral is low-risk; using average funding cost (reflecting bank's full portfolio risk) overstates the true cost. Modigliani-Miller: how you fund should not affect required return.

---

### Questions 7-18 (No solutions)

**7)** A bank's derivatives portfolio with a counterparty has no-default value of $50 million. CVA = $2 million, DVA = $0.5 million. What is the bilateral CVA-adjusted value?

**8)** Describe how a bank would use stored Monte Carlo paths to compute incremental CVA for a new proposed trade.

**9)** If a bank's credit spread doubles, what happens to its DVA? What is the impact on reported profits? Why might this be problematic?

**10)** Compare exposure profiles for matched pairs of (a) interest rate swaps and (b) currency swaps. Which has higher CVA and why?

**11)** Explain how netting affects CVA calculation. Under what circumstances could a new trade with a counterparty reduce CVA?

**12)** Basel III requires market risk capital for CVA exposure to credit spreads but not for CVA exposure to underlying market variables. Why might sophisticated dealers object to this asymmetry?

**13)** A $50M 7-year receiver swap with a BBB corporate (CDS spread 150bp, R=40%). The swap has PV = +$1.2M. Treasury charges SOFR+40bp funding. Capital requirement is $2M with 12% cost of capital. Calculate: (a) CVA using simplified formula, (b) approximate KVA over 7 years, (c) Net desk economics if FVA is $20K.

**14)** Your bank's CDS spread widens from 80bp to 180bp. You have $500M notional of swaps with negative exposure to clients (your liability to them). Calculate the approximate DVA change. Explain why this appears as profit and why you can't realize this gain.

**15)** Compare CVA for two trades with the same BBB counterparty: (a) Payer swap (you pay fixed, receive floating), (b) Receiver swap (you receive fixed, pay floating). In a recession with falling rates, which has higher wrong-way risk? Why?

**16)** A trade has CVA = $500K uncollateralized. With daily margining (10-day MPOR), CVA drops to $50K. But IM posting costs $80K/year (MVA). Compare 5-year economics: uncollateralized vs. cleared.

**17)** Explain the three different notions of XVA "value" (mid-market, accounting, desk economics) and which adjustments are included in each.

**18)** A bank has both a centralized XVA desk and product trading desks. Describe the transfer pricing mechanics: who books what P&L at trade inception, and how does the XVA reserve get released over time?

---

## References

- Hull, *Risk Management and Financial Institutions* (CVA/DVA, exposure simulation, wrong-way risk)
- Hull, *Options, Futures, and Other Derivatives* (CVA/DVA and the “FVA debate” overview)
- Gregory, *The XVA Challenge: Counterparty Credit Risk, Funding, Collateral, and Capital*
- Basel Committee on Banking Supervision, *The Standardised Approach for Counterparty Credit Risk (SA-CCR)* (capital mechanics background)

*Chapters 32–33 cover exposure and collateral mechanics that feed into CVA/FVA; this chapter focuses on the XVA adjustment stack and desk economics.*
