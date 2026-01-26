# Chapter 34: XVA Overview — CVA, DVA, FVA, and the New Derivatives Valuation Landscape

---

## Introduction

When a derivatives desk marks a swap to market, what is the "true" value? Before 2008, the answer seemed obvious: discount expected cashflows at LIBOR, and you had your price. But the credit crisis shattered this simplicity. Counterparties default. Collateral earns interest. Funding has costs. Regulators demand capital. Each of these realities requires an adjustment to the "clean" price—and collectively, these adjustments have spawned an alphabet soup: CVA, DVA, FVA, MVA, KVA. Welcome to the world of XVAs.

Hull (OFD Chapter 9) defines the starting point precisely: "Most of this book is concerned with determining the no-default value of derivatives, that is, the value assuming that neither of the two sides will default. CVA and DVA are adjustments to the no-default value reflecting the possibility of a default by one of the two sides." But as we will see, the adjustments do not stop at credit. Funding costs, margin requirements, and capital charges each demand their own accounting—creating what practitioners call the XVA "stack" that sits atop the clean valuation.

This chapter provides a conceptual and practical overview of XVAs, connecting to the exposure metrics from Chapter 32 and the collateral discounting principles from Chapter 33. We cover:

1. **CVA: Credit Value Adjustment** — the cost of counterparty default
2. **DVA: Debit Value Adjustment** — the controversial "benefit" from your own potential default
3. **FVA: Funding Value Adjustment** — the cost (or benefit) of funding derivative positions
4. **MVA and KVA: Margin and Capital Adjustments** — newer additions to the XVA stack
5. **Wrong-way risk** — when exposure and default become correlated
6. **Practical implementation** — what goes into an XVA calculation

Why should you care? Because XVAs are not just accounting entries—they affect deal economics, hedge ratios, and even whether trades get done. A corporate treasurer who ignores the dealer's CVA charge will be puzzled by the quote. A risk manager who ignores wrong-way risk will underestimate tail exposures. Understanding XVAs is now table stakes for anyone in derivatives markets.

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

Hull (OFD Chapter 9) notes a fundamental divide: "Financial economists have no problem with CVA and DVA, but have reservations about FVA, MVA, and KVA." We will explore why this debate persists.

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

### 34.2.2 The CVA Formula

Hull (RM Chapter 20) provides the discrete-time formula. Suppose the life of the longest derivative is $T$ years, divided into $n$ intervals. Define:

- $q_i$: risk-neutral probability of default during interval $i$
- $v_i$: present value of expected exposure at the midpoint of interval $i$, conditional on default
- $R$: recovery rate

Then:

$$\boxed{\text{CVA} = \sum_{i=1}^{n} (1-R) \, q_i \, v_i}$$

This formula appears "deceptively simple," as Hull notes, but the implementation is "quite complicated and computationally very time-consuming." The challenge lies in computing the $v_i$ terms, which require simulating the entire portfolio forward in time.

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

This formula connects directly to the survival probability framework in Chapter 36, where we derive hazard rates from credit spreads in detail.

**Why risk-neutral probabilities?** Hull emphasizes that "the calculation of CVA involves the valuation of potential future cash flows and... it is correct to use risk-neutral, rather than real-world, default probabilities for valuation." The risk-neutral measure is appropriate because CVA is a price, not a risk metric.

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

### 34.2.6 CVA After Accounting for Defaults

Taking CVA into account, the value of the derivatives portfolio to the bank becomes:

$$f_{\text{nd}} - \text{CVA}$$

This is a **unilateral adjustment**—it reflects only the counterparty's default risk, not the bank's own default risk. As we will see, the full picture requires DVA as well.

---

## 34.3 DVA: Debit Value Adjustment

### 34.3.1 The Controversial "Gain from Default"

The debit (or debt) value adjustment (DVA) is the mirror image of CVA. Hull (RM Chapter 20) defines it: "DVA is the expected cost to the counterparty because the dealer might default. It is the counterparty's CVA. If DVA is a cost to the counterparty, it must be a benefit to the dealer."

The formula is symmetric to CVA:

$$\boxed{\text{DVA} = \sum_{i=1}^{n} (1-R^*) \, q_i^* \, v_i^*}$$

where:
- $q_i^*$: probability of default by the *bank* during interval $i$
- $v_i^*$: present value of the counterparty's exposure to the bank
- $R^*$: recovery rate if the bank defaults

### 34.3.2 Why DVA Exists: The Zero-Sum Argument

The logic for DVA rests on a fundamental principle: "Derivatives are zero-sum games." Hull (OFD Chapter 9) explains: "The gain to one side always equals the loss to the other side. If the bank's counterparty is worse off because of the possibility that the bank will default on the outstanding derivatives, the bank (or the bank's creditors) must be better off."

Without DVA, deals often cannot be agreed. Hull provides an illuminating example:

Consider two parties X and Y negotiating a swap with no-default fair rate of 2.2%.

- X calculates: "Accounting for Y's default risk, I should pay only 2.1%"
- Y calculates: "Accounting for X's default risk, I should receive 2.35%"

If each side considers only the *other's* default risk, there is no overlap—no deal gets done. DVA helps both sides value expected cashflows similarly, enabling trade.

### 34.3.3 Bilateral CVA

Taking both CVA and DVA into account, the value to the bank is:

$$\boxed{V = f_{\text{nd}} - \text{CVA} + \text{DVA}}$$

This is the **bilateral CVA** framework:
- CVA reduces value (cost of counterparty default)
- DVA increases value (benefit from potential own default)

The bilateral framework is now standard in accounting (ASC 820, IFRS 13) and is required for fair value reporting.

### 34.3.4 The Accounting Paradox

Hull (OFD Chapter 9) notes a "counterintuitive effect" of DVA: "As the bank's creditworthiness declines, DVA increases. This makes the derivatives portfolio more valuable to the bank."

When a bank's credit spreads widen:
- $q_i^*$ increases (higher default probability)
- DVA increases
- Reported profits increase

This seems perverse—the worse a bank does, the more profit it reports? The reason is that as default becomes more likely, the bank is "more likely that it will not have to honor its derivatives obligations."

In 2011, some banks reported billions in DVA gains during periods of credit stress. Hull (RM Chapter 20) reports: "Some banks reported several billion dollars of profits from this source in the third quarter of 2011." Regulators responded by excluding DVA gains and losses from the definition of common equity for regulatory capital purposes.

---

## 34.4 Worked Example: CVA for a Forward Contract

**Example 34.2 — Gold Forward CVA (from Hull RM Example 20.3)**

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

**Finance theory:** Hull (OFD Chapter 9) quotes the classic Modigliani-Miller argument: "The way a project is funded should not affect the required return on the investment. The required return should reflect the riskiness of the project."

Margin is low-risk. If we accept it earns only Fed funds + 10bp, then the funding cost is 30bp, not 120bp.

Hull provides a dialogue capturing the essence of the debate:

> **Financial Economist:** The cost you should use for funding a project should reflect the risk of the project.
>
> **Financial Engineer:** But tying up funds in initial margin prevents me from using the funds elsewhere. There is a cost to low-risk, low-return projects.
>
> **Financial Economist:** You talk as though funds are in short supply. If you have good projects, the market will provide funding.
>
> **Financial Engineer:** I am not sure that is how things work in practice.

The correct answer depends on whether one uses *marginal* or *average* funding costs—a debate with no clear resolution. As Hull (OFD Chapter 9) observes, "This argument is over 50 years old in the finance literature and so its theoretical validity has stood the test of time. Many practitioners disagree with the theory."

### 34.6.4 MVA: Margin Value Adjustment

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

### 34.7.2 The Complete XVA Stack

Putting it all together:

$$V_{\text{all-in}} = f_{\text{nd}} - \text{CVA} + \text{DVA} - (\text{FCA} - \text{FBA}) - \text{MVA} - \text{KVA}$$

All XVAs are "computationally time-consuming to calculate. Monte Carlo simulations are necessary to determine expected credit exposures, expected funding costs, and expected capital requirements at future times."

---

## 34.8 Wrong-Way Risk

### 34.8.1 Definition

Hull (RM Chapter 20) defines the key concept: "A situation where there is a positive dependence between the probability of default and the exposure is referred to as **wrong-way risk**. A situation where there is negative dependence is referred to as **right-way risk**."

- **Wrong-way risk:** Default more likely when exposure is high → CVA understated
- **Right-way risk:** Default more likely when exposure is low → CVA overstated

The standard CVA formula assumes independence between $q_i$ and $v_i$. Wrong-way risk means this assumption is violated in a dangerous direction.

### 34.8.2 When Wrong-Way Risk Arises

**Example: CDS Protection Selling**

A counterparty sells credit protection via CDS. When the reference entity's spreads widen:
1. The CDS has positive value to the dealer (exposure increases)
2. The protection seller's credit likely worsens (correlated spreads)
3. Default probability increases exactly when exposure is highest

This is wrong-way risk. AIG is the canonical example—it sold massive amounts of credit protection, and when credit markets deteriorated, both its exposure and its default probability spiked simultaneously.

**Example: Speculating Counterparty**

Hull (RM Chapter 20) notes: "A situation in which a company is speculating by entering into many similar trades with one or more dealers is likely to lead to wrong-way risk for these dealers. This is because the company's financial position and therefore its probability of default is likely to be affected adversely if the trades move against the company."

### 34.8.3 Right-Way Risk

When a counterparty is hedging an existing exposure, right-way risk can occur. Hull explains: "If a company enters into transactions with a dealer to partially hedge an existing exposure, there should in theory be right-way risk. This is because, when the transactions move against the counterparty, it will be benefiting from the unhedged portion of its exposure so that its probability of default should be relatively low."

### 34.8.4 Quantifying Wrong-Way Risk

The CVA formula assumes independence between $q_i$ and $v_i$. Wrong-way risk means this assumption is violated.

Basel II rules use an "alpha" multiplier to address this:
- Minimum alpha = 1.2 (CVA must be at least 20% higher than independence-based model)
- Default alpha = 1.4 (if bank has no own model)

Hull (RM Chapter 20) reports: "Estimates of alpha reported by banks range from 1.07 to 1.10."

More sophisticated approaches model hazard rates as functions of observable market variables. Hull and White (2012) propose a model where "the hazard rate at time $t$ is a function of variables observable at that time." The parameter describing the dependence can be estimated by relating past credit spreads for the counterparty to what the portfolio value would have been historically.

---

## 34.9 CVA Risk Management

### 34.9.1 CVA as a Derivative

Hull (RM Chapter 20) makes a key observation: "A dealer has one CVA for each counterparty. These CVAs can themselves be regarded as derivatives. They are particularly complex derivatives."

When CVA increases, reported income decreases. Dealers hedge CVAs like other derivatives, which requires computing Greek letters for CVA with respect to:
1. **Market variables** affecting $v_i$ (rates, FX, commodities)
2. **Credit spreads** affecting $q_i$

### 34.9.2 CVA Sensitivity to Credit Spreads

From the CVA formula, the sensitivity to a parallel shift $\Delta s$ in credit spreads is (Hull RM equation 20.5):

$$\Delta(\text{CVA}) \approx \sum_{i=1}^{n}\left[t_i e^{-s_i t_i/(1-R)} - t_{i-1} e^{-s_{i-1} t_{i-1}/(1-R)}\right] v_i \, \Delta s + \text{(gamma terms)}$$

Basel III requires market risk capital for CVA risk arising from credit spread changes. However, Hull notes that "risks arising from changes in the market variables affecting the $v_i$ are not included in market risk capital calculations. This is presumably because they are more difficult to calculate."

This asymmetry has drawn criticism: "Sophisticated dealers who are capable of quantifying the $v_i$ risks have complained that, if they hedge these risks, they will be increasing their capital requirements. This is because the hedging trades would be taken into account in determining market risk capital whereas the CVA exposure to the market variables would not."

### 34.9.3 CVA Hedging

Dealers can hedge CVA by:
1. **Buying CDS protection** on counterparties
2. **Hedging market exposures** (rates, FX deltas)
3. **Adjusting deal terms** to reduce exposure

Hull (RM Chapter 20) notes that dealers "sometimes buy protection against their counterparties defaulting using credit default swaps or similar instruments."

---

## 34.10 Worked Example: Simple CVA for a Swap

**Example 34.3 — IRS CVA Calculation**

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

## 34.11 Practical Implementation Notes

### 34.11.1 What Goes Into an XVA System

A production XVA system requires:

1. **Monte Carlo engine:** Simulate all market variables (rates, FX, credit, commodities) under risk-neutral measure to portfolio maturity
2. **Valuation engine:** Price all trades at each simulation time step
3. **Netting logic:** Aggregate values within netting sets
4. **Collateral model:** Apply CSA rules (thresholds, MTA, cure period)
5. **Exposure calculation:** Compute $\max(V - C, 0)$ at each time
6. **Credit data:** Counterparty credit spreads, recovery assumptions
7. **Aggregation:** Sum across time intervals using CVA formula

Hull notes that "dealers may have transactions with thousands of counterparties so that the calculation of the $v_i$ for all of them can be computationally very intensive."

### 34.11.2 Incremental XVA for New Trades

When a new trade is proposed, the desk needs to know its incremental CVA impact. Recalculating the full Monte Carlo is impractical.

Hull (RM Chapter 20) describes the solution: "When the CVA calculations... are carried out, the paths followed by market variables on each simulation trial and the value of the portfolio on each simulation trial are stored. When a potential new transaction is being considered, its value at the future times is calculated for the values of the market variables that were obtained on the simulation trials."

The stored-path approach allows rapid "incremental XVA" pricing for trade approval by computing:
1. New trade's value on each stored path at each time
2. Incremental impact on portfolio value and exposure
3. Incremental effect on CVA via $\sum (1-R) q_i \Delta v_i$

### 34.11.3 Machine Learning Applications

Given the computational intensity, some banks use neural networks to approximate XVA calculations. Hull (OFD Chapter 9) notes: "Once the network has been constructed, calculating the target from the features is very fast."

The approach:
1. Generate many random portfolio configurations
2. Compute XVAs via full Monte Carlo
3. Train neural network to replicate results
4. Use network for real-time incremental XVA

This allows rapid responses to trade inquiries while maintaining accuracy calibrated to full Monte Carlo.

---

## 34.12 Summary

1. **XVAs are adjustments** to the no-default derivative value that account for real-world frictions: credit risk, funding costs, margin, and capital
2. **CVA** = present value of expected loss from counterparty default; reduces derivative value
3. **DVA** = present value of expected gain from own default; increases value but is controversial
4. **Bilateral CVA** = $f_{nd} - \text{CVA} + \text{DVA}$
5. **FVA, MVA, KVA** address funding, margin, and capital costs; theory vs practice debate on whether they should exist
6. **Wrong-way risk** occurs when exposure and default probability are positively correlated
7. **CVA is itself a derivative** that dealers hedge like other complex positions
8. **Implementation** requires Monte Carlo simulation of all market variables and trades
9. **Exposure profiles differ** dramatically by product type—currency swaps have much higher CVA than interest rate swaps of comparable notional

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| CVA | PV of expected loss from counterparty default | Cost of credit risk in derivatives |
| DVA | PV of expected gain from own default | Enables bilateral pricing agreement |
| Unilateral CVA | $f_{nd} - \text{CVA}$ | One-sided credit adjustment |
| Bilateral CVA | $f_{nd} - \text{CVA} + \text{DVA}$ | Two-sided credit adjustment |
| FVA | Adjustment for funding costs/benefits | Captures asymmetric collateralization |
| MVA | Cost of funding initial margin | Growing with margin requirements |
| KVA | Cost of regulatory capital | Links capital to pricing |
| Wrong-way risk | Positive correlation of exposure and PD | CVA understated if ignored |
| Cure period | Time between last margin call and close-out | Creates residual exposure |
| $v_i$ | PV of expected exposure at time $t_i$ | Key input to CVA formula |
| $q_i$ | Risk-neutral default probability in interval $i$ | From counterparty credit spreads |

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
| KVA | Capital Value Adjustment |
| $q_i$ | Risk-neutral probability of counterparty default in interval $i$ |
| $v_i$ | PV of expected exposure at midpoint of interval $i$ |
| $R$ | Recovery rate (fraction recovered in default) |
| $s$ | Credit spread |
| $\bar{\lambda}$ | Average hazard rate (default intensity) |
| $E(t)$ | Exposure at time $t$ |
| $C(t)$ | Collateral at time $t$ |
| $c$ | Cure period / MPOR |

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

### Questions 7-12 (No solutions)

**7)** A bank's derivatives portfolio with a counterparty has no-default value of $50 million. CVA = $2 million, DVA = $0.5 million. What is the bilateral CVA-adjusted value?

**8)** Describe how a bank would use stored Monte Carlo paths to compute incremental CVA for a new proposed trade.

**9)** If a bank's credit spread doubles, what happens to its DVA? What is the impact on reported profits? Why might this be problematic?

**10)** Compare exposure profiles for matched pairs of (a) interest rate swaps and (b) currency swaps. Which has higher CVA and why?

**11)** Explain how netting affects CVA calculation. Under what circumstances could a new trade with a counterparty reduce CVA?

**12)** Basel III requires market risk capital for CVA exposure to credit spreads but not for CVA exposure to underlying market variables. Why might sophisticated dealers object to this asymmetry?

---

## Source Map

### (A) Verified Facts (Source-Backed)

| Fact | Source |
|------|--------|
| CVA is the bank's estimate of PV of expected cost from counterparty default | Hull OFD Ch 9, Hull RM Ch 20 |
| CVA formula: $\text{CVA} = \sum (1-R) q_i v_i$ | Hull RM equation (20.1), Hull OFD equation (9.1) |
| DVA is the counterparty's CVA; a benefit to the bank | Hull OFD Ch 9, Hull RM Ch 20 |
| Bilateral value = $f_{nd} - \text{CVA} + \text{DVA}$ | Hull OFD Ch 9, Hull RM Ch 20 |
| $q_i$ calculated from counterparty credit spreads using hazard rate formula | Hull RM equations (20.2), (20.3) |
| Average hazard rate $\bar{\lambda}_i = s_i/(1-R)$ | Hull RM equation (20.2) |
| $v_i$ computed via Monte Carlo simulation in risk-neutral world | Hull RM Section 20.2 |
| Cure period effect: collateral at default reflects old MTM | Hull RM Example 20.1 |
| FVA = FCA - FBA; adjusts for funding costs | Hull OFD Section 9.2 |
| MVA adjusts for initial margin costs | Hull OFD Section 9.2 |
| KVA adjusts for capital requirements | Hull OFD Section 9.3 |
| Wrong-way risk: positive correlation of exposure and default | Hull RM Section 20.5 |
| Alpha multiplier for wrong-way risk: floor 1.2, default 1.4 (Basel) | Hull RM Section 20.5 |
| Bank estimates of alpha range from 1.07 to 1.10 | Hull RM Section 20.5 |
| CVAs are complex derivatives that dealers hedge | Hull RM Section 20.4 |
| CVA sensitivity to credit spread changes: equation (20.5) | Hull RM Section 20.4 |
| Debate on FVA: average vs marginal funding costs | Hull OFD Section 9.2 |
| DVA excluded from CET1 by regulators | Hull RM Section 20.6, Hull OFD Ch 9 |
| Some banks reported billions in DVA gains in Q3 2011 | Hull RM Section 20.6 |
| Gold forward CVA example calculation | Hull RM Example 20.3 |
| Interest rate swaps have hump-shaped exposure profile | Hull RM Section 20.7 |
| Currency swaps have rising exposure profile due to principal exchange | Hull RM Section 20.7 |
| Incremental CVA via stored Monte Carlo paths | Hull RM Section 20.3 |
| Neural networks used for XVA approximation | Hull OFD Section 9.4 |
| Modigliani-Miller argument on funding costs | Hull OFD Section 9.2 (citing MM 1958, 1963) |
| Dealers may have 1.5 million transactions with thousands of counterparties | Hull RM Section 20.2 (citing Lehman) |

### (B) Reasoned Inference (Derived from A)

- Example 34.3 (IRS CVA) derives from applying the verified CVA formula to illustrative inputs
- Connection to Chapter 33 collateral discounting follows from the shared framework of CSA mechanics
- Connection to Chapter 36 survival probabilities follows from the hazard rate formulas
- Unit check analysis follows from dimensional analysis of the CVA formula

### (C) Flagged Uncertainties

- **I'm not sure** about the exact implementation of FVA in production systems—the theory vs practice debate suggests significant variation across desks
- **I'm not sure** whether MVA/KVA formulas are standardized or desk-specific—the sources note ongoing debate and variation
- The optimal treatment of wrong-way risk beyond the alpha multiplier is not fully specified in the sources; Hull (RM Section 20.5) notes that "more sophisticated approaches" exist but details are limited
- Specific neural network architectures for XVA approximation are not detailed in the sources

---

## References

- Hull, J.C. *Options, Futures, and Other Derivatives*, 11th ed., Chapter 9: "XVAs"
- Hull, J.C. *Risk Management and Financial Institutions*, 5th ed., Chapter 20: "CVA and DVA"
- Gregory, J. *The XVA Challenge: Counterparty Credit Risk, Funding, Collateral, and Capital*. Wiley, 2015.
- Hull, J.C. and A. White. "CVA and Wrong Way Risk," *Financial Analysts Journal* 68, no. 5 (2012).
- Hull, J.C. and A. White. "The FVA Debate," *Risk*, 25th anniversary edition (July 2012).
- Andersen, L.B.G., D. Duffie, and Y. Song. "Funding Value Adjustments," Working Paper, SSRN 2746010.
- Modigliani, F. and M.H. Miller. "The Cost of Capital, Corporation Finance, and the Theory of Investment," *American Economic Review* 48, no. 3 (1958).

---

*Chapter verified against Hull OFD Chapter 9 and Hull RM Chapter 20. All formulas and examples trace to source material.*
