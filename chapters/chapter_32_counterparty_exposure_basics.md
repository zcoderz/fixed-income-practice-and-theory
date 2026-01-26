# Chapter 32: Counterparty Exposure Basics — Netting, Collateral, and Margin Timing

---

## Introduction

When Lehman Brothers filed for bankruptcy on September 15, 2008, it had approximately 1.5 million derivatives transactions outstanding with about 8,000 counterparties. Each of those counterparties faced an immediate question: *How much do they owe us, and how much of that will we ever recover?*

This question—the *exposure* question—is the foundation of counterparty credit risk. Before you can price CVA, stress-test a portfolio, or set a credit limit, you must first answer: what is the potential loss if the counterparty defaults tomorrow? And critically: how does that exposure evolve over time as markets move?

The challenge is that derivatives exposure is unlike loan exposure. A loan's exposure is simply its outstanding principal. But a derivative's exposure is uncertain—it depends on market movements, the structure of the trade, and the web of legal and operational arrangements that govern what happens when things go wrong. As Hull observes in *Risk Management and Financial Institutions*, a dealer's exposure on a derivative "can be zero today and substantial tomorrow, or vice versa"—the bank might be in-the-money or out-of-the-money depending on how rates, FX, or credit spreads move.

This chapter builds the **exposure primitives**—the building blocks you need before tackling CVA, XVA, or any counterparty risk framework:

1. **Exposure as a function of MTM** — Why exposure equals $\max(V, 0)$ and how collateral modifies this
2. **Netting and close-out** — How ISDA master agreements transform counterparty risk and why "netting sets" matter
3. **Collateral mechanics** — VM, IM, thresholds, MTAs, and how they reduce (but never eliminate) exposure
4. **The margin period of risk (MPOR)** — Why even "fully collateralized" trades carry residual exposure
5. **Wrong-way risk** — When exposure and default probability move together (a preview; modeling comes later)

None of this machinery is glamorous. But every CVA calculation, every credit limit decision, and every capital charge traces back to these exposure primitives. Get them wrong, and nothing downstream makes sense.

---

## Conventions and Notation

Before diving into the mechanics, we must establish careful sign conventions. Counterparty exposure calculations are notorious for sign errors—getting the direction of collateral or exposure wrong can reverse your conclusion entirely.

### 32.0.1 Perspective and Sign Convention

We adopt the **bank/dealer perspective** throughout. The bank has a portfolio of derivatives with a counterparty, and we measure the bank's credit risk to that counterparty.

| Convention | Description |
|------------|-------------|
| **MTM sign** | $V(t)$ = net mark-to-market value of all transactions in the netting set at time $t$. **$V(t) > 0$ means the bank is in-the-money** (the counterparty owes the bank). |
| **Collateral sign (Hull-style)** | $C$ = collateral posted by the counterparty to the bank. If $C < 0$, then $-C$ is collateral posted by the bank to the counterparty. |
| **Exposure** | $E = \max(V - C, 0)$ always (bank's exposure to counterparty). |

These conventions follow Hull's treatment in *Risk Management and Financial Institutions*, which provides the clearest operational definitions of exposure and collateral.

### 32.0.2 Symbol Glossary

| Symbol | Definition | Units |
|--------|------------|-------|
| $t$ | Time (years unless stated as "days") | Years |
| $V(t)$ | Net MTM of all trades in a netting set at time $t$ | Currency |
| $C(t)$ | Collateral held by the bank at time $t$ | Currency |
| $E(t)$ | Bank's exposure to the counterparty at time $t$ | Currency |
| $H$ | Threshold for variation margin | Currency |
| $m$ | Minimum transfer amount (MTA) | Currency |
| $c$ | Cure period / MPOR length | Days |
| $q$ | Quantile level (e.g., 95%) | Dimensionless |
| $D(0,t)$ | Discount factor from 0 to $t$ | Dimensionless |
| $R$ | Recovery rate (in $[0,1]$) | Dimensionless |

---

## 32.1 Exposure as a Function of MTM

### 32.1.1 The Basic Exposure Identity

Consider a dealer with a single derivatives transaction outstanding with a counterparty. The value of the transaction to the dealer is $V$. If the counterparty defaults, what does the dealer lose?

If $V < 0$, the dealer owes the counterparty—there is no credit loss (in fact, the dealer benefits from not having to pay). But if $V > 0$, the counterparty owes the dealer, and the dealer becomes an unsecured creditor for the amount $V$.

Hull makes this precise: "The dealer's net exposure is $\max(V, 0)$ where the variable $V$ is the market value of the derivative at the time of the default."

$$\boxed{E(t) = \max(V(t), 0)}$$

> **Analogy: The Casino Chip**
>
> Counterparty Risk is like holding a Casino Chip.
>
> *   **If you are winning (\$1000 chip)**: You have exposure to the Casino. You worry: "Does the cage have enough cash to pay me?"
> *   **If you are losing**: You owe the Casino. You have *no* exposure to them (they have exposure to *you*).
> *   **Max(V, 0)**: You only care about the counterparty's health when you are winning. If you are losing, their bankruptcy doesn't hurt you (in fact, you might get out of paying!).

**This is the fundamental exposure identity** (without collateral). Exposure is the positive part of the portfolio's MTM.

**Unit check:** $V$ is currency $\Rightarrow E$ is currency.

**Sanity checks:**
- If $V(t) \leq 0$, the bank owes the counterparty, so the bank's credit exposure is zero.
- If $V(t) = 0$, the trade is at-the-money, and exposure is zero.
- Exposure is always non-negative.

### 32.1.2 Current Exposure vs. Potential Future Exposure

The exposure identity $E = \max(V, 0)$ can be evaluated at any point in time:

**Current exposure** is the exposure computed from today's MTM—the replacement cost "right now." If you had to close out the trades today, current exposure is what you'd lose (before recovery) if the counterparty defaulted at this instant.

**Potential future exposure (PFE)** recognizes that the counterparty might not default today—they might default next month, or next year. By then, market conditions will have changed, and $V$ could be much larger (or smaller) than today. PFE asks: *How bad could exposure get in the future?*

Hull defines peak exposure as "a high percentile of simulated exposures at each future time (e.g., 97.5%)." The maximum peak exposure is the maximum of these peak exposures across all future times.

The Group of Thirty's risk management recommendations, cited in Hull, emphasize that "credit exposures arising from derivatives trading should be assessed based on the current replacement value of existing positions **and potential future replacement costs**."

This dual assessment—current and potential—is the foundation of credit limit frameworks and regulatory capital calculations.

---

## 32.2 Netting: From Trade-Level to Portfolio-Level Exposure

### 32.2.1 The Legal Basis for Netting

Suppose a bank has 50 different derivatives trades with a counterparty—some in-the-money to the bank, some out-of-the-money. If the counterparty defaults, what happens?

Without netting, the counterparty's bankruptcy trustee might adopt a "cherry-picking" strategy: claim full payment on trades where the bank owes money, while offering only a fractional recovery on trades where the counterparty owes the bank. The bank would have to pay its full obligations but receive only cents on the dollar.

**Netting** prevents this. Hull explains: "Netting in ISDA master agreements means that upon default, all covered transactions are treated as a single transaction." The close-out amount is computed on the net value of all trades—gains on some trades offset losses on others.

This legal protection is not automatic. It requires:
1. A master agreement (typically an ISDA Master Agreement) between the parties
2. A legal opinion confirming enforceability in the relevant jurisdictions
3. Proper documentation and identification of which trades are covered

### 32.2.2 The Mathematics of Netting

Let trades have individual values $V_1, V_2, \ldots, V_N$ at default time.

**Without netting** (trade-by-trade exposure):

$$E_{\text{no net}} = \sum_{i=1}^{N} \max(V_i, 0)$$

Each positive trade value contributes to exposure; negative trade values are ignored (you still owe them, but you're at risk on the positive ones).

**With netting** (net portfolio exposure):

The "single transaction" principle means the close-out is computed on the net value:

$$V_{\text{net}} = \sum_{i=1}^{N} V_i$$

$$\boxed{E_{\text{net}} = \max\left(\sum_{i=1}^{N} V_i, 0\right)}$$

### 32.2.3 The Netting Benefit

The key mathematical result is that netting can only reduce exposure:

$$\boxed{\max\left(\sum_{i=1}^{N} V_i, 0\right) \leq \sum_{i=1}^{N} \max(V_i, 0)}$$

This follows from the subadditivity of the $\max(\cdot, 0)$ function. The inequality is strict whenever some trades are positive and some are negative—exactly the situation where gains offset losses.

**Perfect offset example:** Suppose trade 1 has $V_1 = +10$ and trade 2 has $V_2 = -10$ (perfectly offsetting). Without netting, exposure is $\max(10,0) + \max(-10,0) = 10$. With netting, exposure is $\max(0,0) = 0$. The netting benefit is complete.

**Imperfect offset example:** Suppose $V_1 = +10$ and $V_2 = -8$. Without netting, exposure is $10 + 0 = 10$. With netting, exposure is $\max(2,0) = 2$. Netting reduces exposure from 10 to 2.

### 32.2.4 Netting Sets in Practice

The term **netting set** refers to the collection of trades whose close-out amount is computed on a net basis and whose collateral is managed together under a single CSA.

This creates an important operational constraint: "Credit exposures to a counterparty should be aggregated in a way that reflects enforceable netting agreements" (Group of Thirty recommendations, cited in Hull).

You cannot mix trades from different legal entities, or trades under different master agreements, into a single netting set. And if netting is not legally enforceable in the counterparty's jurisdiction, you must use trade-by-trade exposure—even if you have an ISDA in place.

---

## 32.3 Collateral and Margin

### 32.3.1 How Collateral Modifies Exposure

When collateral is posted, the exposure identity becomes:

$$\boxed{E = \max(V - C, 0)}$$

where $C$ is collateral held by the bank (using our sign convention).

Hull provides a careful interpretation of this formula:
- If $V > 0$ and the counterparty has posted collateral $C > 0$, then exposure shrinks to $\max(V - C, 0)$.
- If the bank has posted collateral ($C < 0$), this collateral may be lost on counterparty default. If the bank's posted collateral exceeds what it owed ($-V$), the "excess posted" becomes an exposure.

**Example:** The bank's trades are worth $V = -50$ (bank owes counterparty). The bank has posted $C = -55$ in collateral (bank posted 55 to counterparty). If the counterparty defaults, the bank's exposure is $\max(-50 - (-55), 0) = \max(5, 0) = 5$. The bank loses 5 because it over-posted by 5 and won't get that back.

### 32.3.2 Variation Margin (VM)

**Variation margin** is collateral that tracks the current positive value of transactions to the other side. Hull explains that VM can be set so that "when transactions are positive to one side, the other posts collateral equal to that positive value."

In a **perfect, instantaneous, zero-threshold VM world**, current exposure would be driven to zero. If $V$ changes, collateral immediately adjusts so that $C = V$ when $V > 0$ (or $C = V$ for two-way VM). Then $E = \max(V - V, 0) = 0$.

In reality, this idealized state is never achieved because of:
1. **Thresholds** — collateral only required above a threshold
2. **Minimum transfer amounts (MTAs)** — small moves don't trigger transfers
3. **Timing lags** — collateral reflects yesterday's (or earlier) MTM, not today's
4. **Cure period / MPOR** — between default and close-out, collateral is stale

### 32.3.3 Initial Margin (IM) and Independent Amounts

**Initial margin** is a buffer posted at trade inception (or when exposure grows) to cover potential future exposure during the margin period of risk.

Hull notes that for bilaterally cleared transactions subject to post-crisis regulations, "IM must cover market moves over a 10-day stressed period with 99% confidence, and is typically segregated/held by a custodian."

In bilateral OTC, IM is often called an **independent amount**. The purpose is the same: protect against exposure that emerges during the gap between the last collateral call and close-out.

> **Connection to discounting:** The presence of collateral affects not just exposure but also *how we discount* derivative cashflows. When collateral earns the overnight rate, OIS discounting becomes appropriate. This connection between collateral and discounting is developed fully in **Chapter 33**.

### 32.3.4 Thresholds and MTAs

A **threshold** means VM is required only above a certain amount. Hull's example: "if value is 9 and threshold is 10, no collateral; if value is 11, collateral of 1."

The threshold creates a **residual unsecured band**—even with VM in place, exposure up to the threshold level remains uncollateralized.

The **minimum transfer amount (MTA)** avoids operational "nuisance" transfers for small amounts. If the required transfer is less than the MTA, no transfer occurs.

Both features increase residual exposure but reduce operational friction.

### 32.3.5 Rehypothecation

**Rehypothecation** refers to the practice where a party that receives collateral reuses that collateral to meet its own margin requirements elsewhere. Hull defines it as "the use of collateral posted by one counterparty to satisfy the collateral requirements of another counterparty."

The practice improves market liquidity but introduces additional risk. If the party holding your collateral defaults, you may not get that collateral back—particularly if it has been rehypothecated to third parties. Hull notes that "after Lehman Brothers declared bankruptcy in September 2008, clients (particularly European hedge fund clients) found it difficult to get a return of the collateral they had posted with Lehman because it had been rehypothecated."

As a result, post-crisis regulations and CSA amendments now commonly include clauses banning or limiting rehypothecation. Initial margin under bilateral clearing regulations must typically be segregated with a third-party custodian precisely to prevent rehypothecation.

### 32.3.6 Downgrade Triggers

CSAs may include **downgrade triggers**—clauses stating that if one party's credit rating falls below a specified level, the other party can demand additional collateral or terminate all transactions.

Hull provides the AIG example: "Many of AIG's transactions stated that AIG did not have to post collateral provided its credit rating remained above AA. However, once it was downgraded below AA, collateral was required." When AIG was downgraded by all three rating agencies on September 15, 2008, this triggered massive collateral calls that AIG could not meet, ultimately requiring a government bailout.

Downgrade triggers are a double-edged sword:
- They provide some protection against counterparty deterioration
- But they do not protect against sudden "jump-to-default" events (e.g., AA to default)
- If a company has many downgrade triggers across its contracts, a single downgrade can trigger cascade effects—precisely when the company is least able to respond

---

## 32.4 The Margin Period of Risk (MPOR)

### 32.4.1 Why MPOR Matters

Even with daily variation margin and zero thresholds, exposure is not zero. The reason is timing: there is a gap between when the counterparty stops posting collateral and when trades are actually closed out.

Hull defines the **cure period** (also called margin period of risk or MPOR) as "the time between the counterparty stopping collateral posting and close-out; typically 10 or 20 days."

During this period:
1. The counterparty is in distress and stops responding to margin calls
2. The bank must recognize the default, invoke early termination
3. The bank must value the portfolio and execute close-out
4. Throughout, the market is moving—but collateral is frozen

### 32.4.2 Collateral at Default Is Stale

The key implication: **collateral at default reflects the portfolio value $c$ days earlier**, not the current value. Hull emphasizes that "the Monte Carlo simulation to calculate the $v_i$ must be structured so that the value of the derivatives portfolio with the counterparty is calculated at times $t_i^* - c$ as well as at time $t_i^*$."

This stale-collateral effect means that even under "perfect collateralization," exposure can emerge from market moves during MPOR.

### 32.4.3 Hull's MPOR Example

Hull provides a concrete illustration (Example 20.1 in *Risk Management and Financial Institutions*):

> There is a two-way zero threshold collateral agreement between a bank and its counterparty. The cure period is 20 days.
>
> 1. On a particular simulation trial, the value of outstanding transactions to the bank at time $\tau$ is 50 and their value 20 days earlier is 45. The calculation assumes that the bank has collateral worth 45 in the event of a default at time $\tau$. The bank's exposure is therefore 5.
>
> 2. On a particular simulation trial, the value of outstanding transactions to the bank at time $\tau$ is 50 and their value 20 days earlier is 55. In this case, the bank will have adequate collateral and its exposure is zero.

The first scenario shows how MPOR creates exposure: the portfolio moved from 45 to 50 during the cure period, but collateral was stuck at 45.

---

## 32.5 Exposure Metrics: EE, PFE, and EPE

### 32.5.1 Expected Exposure (EE)

**Expected exposure at time $t$** is the expected value of exposure at that time:

$$\boxed{EE(t) := \mathbb{E}[E(t)]}$$

This is the average exposure across all possible market scenarios at time $t$. EE is the exposure measure that enters CVA calculations because CVA is an expected loss, and expected loss involves expected exposure.

### 32.5.2 Potential Future Exposure (PFE)

**Potential future exposure** is a quantile of the exposure distribution:

$$\boxed{PFE_q(t) := \inf\{x : P(E(t) \leq x) \geq q\}}$$

The $q$-quantile (e.g., 95% or 97.5%) answers: "What exposure level will we not exceed with probability $q$?"

Hull describes peak exposure as "a high percentile of simulated exposures at each future time." For example, with 10,000 Monte Carlo trials and a 97.5% percentile, the peak exposure at time $t$ is the 250th highest exposure recorded at that time.

**PFE vs. EE:** PFE answers "how bad can it get?" while EE answers "what's the average?" PFE is used for credit limits and stress testing; EE feeds into CVA pricing.

### 32.5.3 Expected Positive Exposure (EPE)

**EPE** is a time-aggregated measure of expected exposure. The common convention is a time-weighted average:

$$EPE := \frac{1}{T} \int_0^T EE(t) \, dt \quad \text{or discretized:} \quad EPE \approx \sum_i w_i \, EE(t_i)$$

EPE provides a single-number summary of the exposure profile over the life of the portfolio. It is used in regulatory capital calculations and as an input to certain CVA approximations.

> **Note on uncertainty:** The precise definition of EPE (time-weighting, discounting, whether collateral is included) varies across desks and regulatory frameworks. To be certain, verify your desk's specific definition.

### 32.5.4 Risk-Neutral vs. Real-World Simulation

Hull flags an important conceptual distinction: CVA is a valuation exercise requiring risk-neutral simulation, but PFE is scenario analysis asking "how bad can it get?"

"When we calculate peak exposure, we are carrying out a scenario analysis... For this purpose, we should in theory simulate the behavior of market variables in the real world, not the risk-neutral world."

In practice, this distinction is often ignored, and the same risk-neutral simulations are used for both purposes. But understanding the conceptual difference matters for interpreting results.

---

## 32.6 Exposure Profiles for Different Instruments

### 32.6.1 Interest Rate Swaps vs. Currency Swaps

Hull provides a striking comparison (Figure 20.1 in *Risk Management and Financial Institutions*):

> "The expected exposure on the interest rate swaps starts at zero, increases, and then decreases. By contrast, expected exposure on the currency swaps increases steadily with the passage of time."

Why the difference? For interest rate swaps, early in the trade there's much to be exchanged (many future cashflows), so exposure can grow as rates move. But toward the end, very little remains to be exchanged, so exposure declines. The profile is "hump-shaped."

For currency swaps, principals are exchanged at the end of the life. The uncertainty about the exchange rate at maturity drives exposure higher as maturity approaches. The profile is monotonically increasing.

This has practical implications: "The impact of default risk for a dealer in currency swaps is therefore much greater than for a dealer in interest rate swaps."

> **Visual: The Exposure Shape**
>
> *   **Interest Rate Swap ("The Hump")**: Starts low (Par), grows as rates move, then falls to zero as payments are made and time runs out. Peak risk is in the middle.
> *   **FX Swap / Cross-Currency ("The Ramp")**: Starts low, but grows steadily because the huge principal exchange at the end gets riskier and riskier as FX drifts further from the strike. Peak risk is at the very end.

### 32.6.2 Forward Contracts

For a forward contract to buy an asset at price $K$ with current forward price $F_0$, Hull derives the present value of expected exposure:

$$v_i = e^{-rT}\left[F_0 N(d_1) - K N(d_2)\right]$$

where $d_1$ and $d_2$ are Black-Scholes-type quantities. The exposure looks like a call option on the forward price—which makes sense, because the bank's positive exposure occurs when the forward price exceeds the strike.

---

## 32.7 Wrong-Way Risk (Preview)

### 32.7.1 Definition and Intuition

**Wrong-way risk** occurs when exposure and the probability of default are positively correlated. Hull defines it as the situation where "exposure rises when counterparty becomes more likely to default."

The intuition is captured by the phrase: "The counterparty is weakest exactly when I'm most exposed."

### 32.7.2 Examples of Wrong-Way Risk

Hull provides several examples:

**Speculating counterparty:** "A situation in which a company is speculating by entering into many similar trades with one or more dealers is likely to lead to wrong-way risk for these dealers. This is because the company's financial position and therefore its probability of default is likely to be affected adversely if the trades move against the company."

**CDS seller:** A counterparty selling credit protection on a reference entity may experience wrong-way risk for the protection buyer. If the reference entity defaults, the protection becomes very valuable—exactly when the protection seller (who has similar credit characteristics) may also be in distress.
>
> **Classic Example: The Put Option on the Bank**
> If you buy a Put Option on Bank A *from* Bank A...
> *   **The Payoff**: You make money if Bank A's stock crashes.
> *   **The Catch**: If Bank A's stock crashes, Bank A might be bankrupt.
> *   **Result**: You are "winning" exactly when the person paying you is "dying." Your expected recovery is terrible. This is extreme Wrong Way Risk.

### 32.7.3 Right-Way Risk

The opposite is **right-way risk**: exposure and default probability are negatively correlated.

Hull notes: "If a company enters into transactions with a dealer to partially hedge an existing exposure, there should in theory be right-way risk. This is because, when the transactions move against the counterparty, it will be benefiting from the unhedged portion of its exposure so that its probability of default should be relatively low."

> **Scope note:** Modeling wrong-way risk requires specifying the dependence between market variables and default probability. This is a separate (and difficult) modeling problem, treated in later chapters. Here we only preview the concept.

---

## 32.8 CVA Preview: How Exposure Metrics Feed Into Valuation

This chapter focuses on exposure primitives. The full treatment of CVA, DVA, and other valuation adjustments belongs in **Chapter 34 (XVA Overview)**. The connection between collateral and discount curves is developed in **Chapter 33 (Collateral Discounting and OIS)**. But it is useful here to see how exposure connects to valuation.

Hull gives the discretized CVA formula:

$$\boxed{CVA = \sum_{i=1}^{n}(1-R) q_i v_i}$$

where:
- $q_i$ = risk-neutral probability of default during interval $i$ (from credit spreads)
- $v_i$ = present value of expected net exposure at the midpoint of interval $i$
- $R$ = recovery rate

The exposure work in this chapter is about computing the $v_i$. The credit work (survival probabilities, hazard rates) is about computing the $q_i$. CVA combines them.

**Preview approximation:** If we approximate $v_i \approx D(0, t_i) \cdot EE(t_i)$ and $q_i \approx \Delta PD(t_i)$, then:

$$CVA \approx (1-R) \sum_i D(0, t_i) \cdot EE(t_i) \cdot \Delta PD(t_i)$$

This is a desk-friendly approximation of Hull's $(1-R) q_i v_i$ structure.

---

## 32.9 Worked Examples

The following examples build intuition for how exposure, netting, and collateral interact. All use the bank perspective with $E = \max(V - C, 0)$.

### Example A: Single Trade Exposure Distribution

**Setup:** One derivative with future MTM at $t = 1$ year:
- $V = +5$ with probability $0.5$
- $V = -5$ with probability $0.5$

No collateral.

**Step 1: Compute exposure outcomes**
- If $V = +5$: $E = \max(5, 0) = 5$
- If $V = -5$: $E = \max(-5, 0) = 0$

Exposure is $\{5, 0\}$ with probabilities $\{0.5, 0.5\}$.

**Step 2: Expected exposure**

$$EE(1y) = 0.5 \times 5 + 0.5 \times 0 = 2.5$$

**Step 3: PFE at 95%**

CDF: $P(E \leq 0) = 0.5$, $P(E \leq 5) = 1.0$.

The 95% quantile is 5, so $PFE_{95\%}(1y) = 5$.

**Interpretation:** Expected exposure is 2.5, but the 95% worst case is 5—twice as high. PFE captures tail risk that EE averages away.

---

### Example B: Current Exposure vs. Future PFE

**Setup:** Today $t = 0$, MTM is $V(0) = +1$ (no collateral). At $t = 1y$, distribution is as in Example A.

**Current exposure (today):**
$$E(0) = \max(1, 0) = 1$$

**Future PFE (1 year):** From Example A, $PFE_{95\%}(1y) = 5$.

**Interpretation:** Current exposure is small (1), but the 95% "how bad can it get in a year?" measure is 5. This aligns with the Group of Thirty's recommendation to assess both current replacement value and potential future replacement costs.

---

### Example C: Netting Benefit

**Setup:** Two trades at $t = 1y$ with perfectly negative dependence:
- **Scenario S1** (prob 0.5): $V_1 = +10$, $V_2 = -8$
- **Scenario S2** (prob 0.5): $V_1 = -10$, $V_2 = +8$

**Without netting (trade-by-trade):**

$$E_{\text{no net}} = \max(V_1, 0) + \max(V_2, 0)$$

- S1: $10 + 0 = 10$
- S2: $0 + 8 = 8$

$EE_{\text{no net}} = 0.5 \times 10 + 0.5 \times 8 = 9$

$PFE_{95\%} = 10$

**With netting:**

Net MTM $V_{\text{net}} = V_1 + V_2$

- S1: $V_{\text{net}} = 2 \Rightarrow E_{\text{net}} = 2$
- S2: $V_{\text{net}} = -2 \Rightarrow E_{\text{net}} = 0$

$EE_{\text{net}} = 0.5 \times 2 + 0.5 \times 0 = 1$

$PFE_{95\%} = 2$

**Netting benefit:** EE drops from 9 to 1 (89% reduction). PFE drops from 10 to 2 (80% reduction). The offsetting positions almost fully hedge each other from a credit exposure perspective.

---

### Example D: Idealized Zero-Threshold VM

**Setup:** Same as Example A, but assume two-way zero-threshold VM with no lag—collateral always equals $\max(V, 0)$.

At $t = 1y$:
- If $V = +5$: counterparty posts $C = 5$, so $E = \max(5 - 5, 0) = 0$
- If $V = -5$: bank posts 5, so $C = -5$, and $E = \max(-5 - (-5), 0) = 0$

**Result:** Current exposure is eliminated in this idealized setting.

**What remains:** In reality, MPOR makes $C$ stale, and exposure can re-emerge even with "daily VM."

---

### Example E: Threshold and MTA

**Setup:** Same distribution as Example A, but with collateral terms:
- Threshold $H = 2$
- MTA $m = 1$
- Two-way VM, no lag

**Collateral rule:** Call amount $= \max(V - H, 0)$. If call amount $< m$, no transfer.

**Case 1: $V = +5$**
- Call amount $= \max(5 - 2, 0) = 3$
- $3 \geq 1 \Rightarrow C = 3$
- Exposure $E = \max(5 - 3, 0) = 2$

**Case 2: $V = -5$**
- Bank exposure is $\max(-5 - C, 0) = 0$ under symmetric posting

**Results:** $E \in \{2, 0\}$ with equal probability.

$EE = 0.5 \times 2 + 0.5 \times 0 = 1$

$PFE_{95\%} = 2$

**Interpretation:** The threshold creates a "residual unsecured band" of up to 2, even with VM in place.

---

### Example F: Margin Call Lag (Stale Collateral)

**Setup:** VM is called daily but settles with a 1-day lag. Collateral at day $t$ equals yesterday's MTM if positive.

**3-day MTM path:**
- Day 0: $V_0 = 0$
- Day 1: $V_1 = 4$
- Day 2: $V_2 = 6$
- Day 3: $V_3 = 1$

**Collateral:** $C_t = \max(V_{t-1}, 0)$

**Daily exposure:** $E_t = \max(V_t - C_t, 0)$

| Day | $V_t$ | $C_t$ | $E_t$ |
|-----|-------|-------|-------|
| 1 | 4 | 0 | 4 |
| 2 | 6 | 4 | 2 |
| 3 | 1 | 6 | 0 |

**Timing-gap risk:** The lag produces a peak exposure of 4 on Day 1 despite "daily VM." The faster the MTM moves, the larger the lag-induced exposure.

---

### Example G: MPOR and IM Sizing

**Motivation:** IM is described as protecting over a 10-day stressed period with 99% confidence.

**Setup:** Daily MTM change $\Delta V \in \{+1, -1\}$ with probability 0.5 each. Over MPOR = 10 days:

$$\Delta V_{10} = \sum_{j=1}^{10} \Delta V_j$$

This is a symmetric random walk. Define $\Delta V_{10}^+ = \max(\Delta V_{10}, 0)$.

**CDF of $\Delta V_{10}^+$:**
- at 6: 98.93%
- at 8: 99.90%

**99% quantile of $\Delta V_{10}^+$ is 8.**

**IM proxy:** $IM \approx 8$ currency units covers the 99th percentile of 10-day exposure increase.

---

### Example H: Netting + Collateral Combined

**Setup:** 3 trades in one netting set. Two scenarios (each prob 0.5). Collateral terms: threshold $H = 1$, MTA $m = 0.5$, no lag.

**Trade MTMs:**

| Time | Scenario | $V_1$ | $V_2$ | $V_3$ | Net $V$ |
|------|----------|-------|-------|-------|---------|
| $t_1$ | S1 | 3 | -1 | 2 | 4 |
| $t_1$ | S2 | -2 | 1 | -1 | -2 |
| $t_2$ | S1 | 4 | -2 | 1 | 3 |
| $t_2$ | S2 | -3 | 2 | -2 | -3 |
| $t_3$ | S1 | 2 | -1 | -1 | 0 |
| $t_3$ | S2 | 1 | 1 | -1 | 1 |

**Collateral rule:** If $V > H$, counterparty posts $C = V - H$.

**Results:**

| Time | Scenario | $V$ | $C$ | $E$ |
|------|----------|-----|-----|-----|
| $t_1$ | S1 | 4 | 3 | 1 |
| $t_1$ | S2 | -2 | -1 | 0 |
| $t_2$ | S1 | 3 | 2 | 1 |
| $t_2$ | S2 | -3 | -2 | 0 |
| $t_3$ | S1 | 0 | 0 | 0 |
| $t_3$ | S2 | 1 | 0 | 1 |

At each time: $EE = 0.5$, $PFE_{95\%} = 1$

**Interpretation:** The threshold creates a floor on exposure at or below $H$, while netting reduces net MTM volatility by offsetting trades.

---

### Example I: Collateral Frequency Comparison

**Setup:** Same MTM path over 10 days. Compare daily VM (immediate) vs. weekly VM (update at days 0, 5, 10). Zero threshold, no IM.

**Daily MTM:** $V = [0, 1, 3, 2, 5, 4, 6, 3, 2, 1, 0]$

**Daily VM (immediate):** $C_t = V_t$ always, so $E_t = 0$ for all $t$.
- Peak exposure: 0
- Average exposure: 0

**Weekly VM:** Collateral updates only at days 0, 5, 10.

Days 1-4: $C = 0$, so $E = [1, 3, 2, 5]$

Day 5: $C = 4$, so $E_5 = 0$

Days 6-10: $C = 4$, so $E = [2, 0, 0, 0, 0]$

- Peak exposure: 5
- Average exposure: $(1+3+2+5+0+2)/10 = 1.3$

**Interpretation:** Less frequent VM increases both peak and average exposure—purely through timing.

---

### Example J: Cross-Netting Set Impact

**Setup:** Two trades with $V_A = +6$ (always) and $V_B = -5$ (always).

**Case 1: One netting set**

$V_{\text{net}} = 6 - 5 = 1 \Rightarrow E = 1$

**Case 2: Two separate netting sets**

$E_1 = \max(6, 0) = 6$

$E_2 = \max(-5, 0) = 0$

Total $E = 6$

**Fragmentation cost:** Exposure increases from 1 to 6 solely due to netting set boundaries. This illustrates why consolidating trades under a single master agreement is valuable for credit risk.

---

### Example K: CVA Preview Calculation

**Setup:** Discretized CVA with three intervals.
- Times: $t_1 = 1$, $t_2 = 2$, $t_3 = 3$ years
- Recovery $R = 40\%$, so $(1-R) = 0.6$
- Flat discount rate $r = 2\%$
- Expected exposures: $EE = [5, 4, 3]$ million
- Default probability increments: $\Delta PD = [1\%, 1.5\%, 2\%]$

**Discount factors:**
- $D_1 = e^{-0.02} = 0.9802$
- $D_2 = e^{-0.04} = 0.9608$
- $D_3 = e^{-0.06} = 0.9418$

**Compute each term $D \cdot EE \cdot \Delta PD$:**
- Term 1: $0.9802 \times 5 \times 0.01 = 0.04901$
- Term 2: $0.9608 \times 4 \times 0.015 = 0.05765$
- Term 3: $0.9418 \times 3 \times 0.02 = 0.05651$

**Sum:** $0.04901 + 0.05765 + 0.05651 = 0.16317$

**CVA:** $0.6 \times 0.16317 = 0.0979$ million = **\$97,900**

---

## 32.10 Practical Notes

### 32.10.1 Common Pitfalls

**Computing exposure trade-by-trade instead of at netting set level:** This violates netting logic and overstates exposure. Always aggregate within enforceable netting agreements.

**Forgetting timing lags and MPOR:** Collateral is stale by $c$ days. Exposure can appear even under perfect collateralization if MTM moves during the cure period.

**Confusing PFE with EE:** PFE (quantile) answers "how bad?"; EE (mean) answers "on average." Use the right measure for the right purpose—PFE for limits, EE for CVA.

**Sign convention errors:** Always verify that positive $V$ means in-the-money for the bank, and that collateral $C$ follows the correct convention. Sign errors reverse conclusions.

### 32.10.2 Implementation Considerations

**Time grid choices:** Exposure profiles depend on discretization. Finer grids near the front end (where exposure changes rapidly) may be needed.

**Collateral modeling:** Frequency, thresholds, MTAs, and "stale collateral" assumptions can dominate exposure outcomes. Model these carefully.

**Simulation requirements:** Hull notes that dealers may have transactions with thousands of counterparties, making CVA calculation computationally intensive. Efficient simulation and storage of paths is essential.

### 32.10.3 Sanity Checks

- **Netting cannot increase exposure:** If $E_{\text{net}} > E_{\text{no net}}$ in your model, something is wrong.
- **Tighter collateral reduces exposure:** Lower thresholds, lower MTAs, and more frequent VM should all reduce exposure metrics.
- **PFE ≥ EE** for typical distributions. Equality occurs only for degenerate cases (constant exposure).

---

## Summary

Counterparty exposure is the foundation of counterparty credit risk. Before you can price CVA or set credit limits, you must understand how exposure arises from derivatives positions and how it is modified by netting and collateral.

**Key takeaways:**

1. **Exposure identity (no collateral):** $E(t) = \max(V(t), 0)$ where $V$ is net MTM.

2. **Collateralized exposure:** $E = \max(V - C, 0)$ with careful sign conventions.

3. **Netting reduces exposure:** With netting, $E_{\text{net}} = \max(V_{\text{net}}, 0)$ instead of $\sum \max(V_i, 0)$.

4. **VM tracks current MTM** but thresholds and MTAs leave residual exposure.

5. **IM buffers MPOR risk:** Covers exposure change during the cure period.

6. **MPOR makes collateral stale:** Even "perfect" VM leaves exposure from market moves during the 10-20 day cure period.

7. **PFE and EE are different:** PFE is a quantile (tail risk); EE is a mean (for CVA).

8. **Exposure profiles differ by product:** Interest rate swaps have hump-shaped profiles; currency swaps increase monotonically.

9. **Wrong-way risk is dangerous:** Exposure rising exactly when default probability rises amplifies losses.

10. **CVA uses exposure primitives:** $CVA = \sum (1-R) q_i v_i$ where $v_i$ comes from exposure modeling.

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **MTM** $V(t)$ | Net mark-to-market value of netting set | The input to exposure calculation |
| **Exposure** $E(t)$ | $\max(V-C, 0)$—positive part of (MTM minus collateral) | What you lose at counterparty default |
| **EE** | $\mathbb{E}[E(t)]$—expected exposure | Feeds into CVA calculation |
| **PFE** | Quantile of $E(t)$—"how bad can it get?" | Used for credit limits |
| **EPE** | Time-average of EE | Single-number exposure summary |
| **Netting** | Legal aggregation under master agreement | Reduces exposure via offset |
| **VM** | Collateral tracking current MTM | Reduces current exposure |
| **IM** | Buffer for MPOR period | Covers 10-day stressed moves |
| **MPOR** | Cure period (10-20 days) | Makes collateral stale |
| **Threshold** | Level below which no VM required | Creates residual unsecured band |
| **MTA** | Minimum transfer amount | Avoids nuisance transfers |
| **Rehypothecation** | Reuse of received collateral for other purposes | Creates recovery risk if holder defaults |
| **Downgrade trigger** | CSA clause demanding collateral on rating downgrade | Protection vs. cascade risk |
| **Wrong-way risk** | Exposure ↑ when default probability ↑ | Amplifies losses |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What does $V(t)$ represent in counterparty exposure? | Net MTM of all trades in a netting set at time $t$. |
| 2 | What is the bank's exposure without collateral? | $E(t) = \max(V(t), 0)$ |
| 3 | What is the bank's exposure with collateral? | $E = \max(V - C, 0)$ |
| 4 | What is netting (contractually)? | A clause treating all covered trades as a single transaction on default. |
| 5 | How do you compute exposure without netting? | $\sum \max(V_i, 0)$ |
| 6 | How do you compute exposure with netting? | $\max(\sum V_i, 0)$ |
| 7 | What is variation margin (VM)? | Collateral posted to track positive MTM to the other side. |
| 8 | What is another name for IM in bilateral OTC? | Independent amount. |
| 9 | What is the cure period / MPOR? | Delay (typically 10-20 days) between collateral cessation and close-out. |
| 10 | Why does MPOR matter? | Collateral at default reflects value $c$ days earlier, creating exposure if MTM moves. |
| 11 | What is peak exposure? | High percentile of simulated exposures at each future time. |
| 12 | What measure should be used for peak exposure simulation? | Real-world (for scenario analysis), though risk-neutral is often used in practice. |
| 13 | Define EE(t). | $EE(t) = \mathbb{E}[E(t)]$—expected exposure at time $t$. |
| 14 | Define PFE. | The $q$-quantile of $E(t)$—e.g., 95th percentile of exposure. |
| 15 | What is a threshold in a CSA? | Level below which no VM is required. |
| 16 | What is MTA? | Minimum transfer amount—avoids small collateral movements. |
| 17 | What is IM calibrated to cover (post-crisis rules)? | 10-day stressed period at 99% confidence. |
| 18 | How is IM typically held? | Segregated with a third-party custodian. |
| 19 | What is wrong-way risk? | Exposure rising when counterparty default probability rises. |
| 20 | What does "aggregate exposures reflecting netting" mean? | Compute exposure on netted portfolios under enforceable agreements. |
| 21 | What is current exposure? | Replacement cost if counterparty defaulted now. |
| 22 | What is potential future exposure? | Distribution of exposure at future times. |
| 23 | When can bank exposure be positive even if bank posted collateral? | If posted collateral exceeds what bank owed and isn't returned on default. |
| 24 | What is rehypothecation? | Reuse of posted collateral to meet other collateral needs. |
| 25 | Why can't CVA be calculated transaction-by-transaction? | Netting means exposure depends on the net portfolio, not individual trades. |
| 26 | How do interest rate swap exposure profiles behave? | Hump-shaped: start at zero, increase, then decrease. |
| 27 | How do currency swap exposure profiles behave? | Monotonically increasing (principal exchanged at end). |
| 28 | In Example A, what is EE? | 2.5 |
| 29 | In Example A, what is PFE at 95%? | 5 |
| 30 | In Example C, what is netted EE? | 1 |
| 31 | In Example C, what is unnetted EE? | 9 |
| 32 | What is the core operational driver of exposure under VM? | Timing—how quickly collateral updates vs. MTM moves. |
| 33 | For MPOR simulation, what times must be computed? | Values at both $t_i^* - c$ and $t_i^*$. |
| 34 | What do thresholds do to residual exposure? | Create residual unsecured bands. |
| 35 | What is the CVA formula structure? | $CVA = \sum (1-R) q_i v_i$ |
| 36 | What risk does rehypothecation create? | If the party holding collateral defaults, rehypothecated collateral may not be returned. |
| 37 | What happened to Lehman's hedge fund clients due to rehypothecation? | They found it difficult to recover collateral that Lehman had rehypothecated. |
| 38 | What is a downgrade trigger? | CSA clause allowing collateral demands or termination if credit rating falls below a level. |
| 39 | What happened to AIG due to downgrade triggers? | Downgrade below AA triggered massive collateral calls, requiring government bailout. |
| 40 | Why can downgrade triggers be dangerous for the protected party? | They don't protect against sudden jump-to-default events. |

---

## Mini Problem Set

**Instructions:** Problems increase in difficulty. Solution sketches provided for Q1-Q9.

---

**Q1 (Easy):** Single trade: $V \in \{-2, +6\}$ with probs $(0.7, 0.3)$. Compute $E$, $EE$, $PFE_{90}$.

> **Sketch:** $E = \{0, 6\}$. $EE = 0.3 \times 6 = 1.8$. $PFE_{90} = 6$ (CDF at 0 is 0.7 < 0.9).

---

**Q2 (Easy):** If today $V(0) = -4$ and no collateral, what is current exposure?

> **Sketch:** $E(0) = \max(-4, 0) = 0$.

---

**Q3 (Easy):** Two trades with values $V_1 = +3$, $V_2 = -1$. Compute exposure with and without netting.

> **Sketch:** No netting: $3 + 0 = 3$. With netting: $\max(2, 0) = 2$.

---

**Q4 (Easy):** Threshold $H = 5$: if net $V = 7$, what VM is called and what residual exposure remains?

> **Sketch:** Call $= 7 - 5 = 2$. Exposure $= 7 - 2 = 5$.

---

**Q5 (Easy):** Cure period $c$: if $V(t) = 12$ but $V(t - c) = 9$ under zero-threshold VM, what is exposure at $t$?

> **Sketch:** Collateral $C = 9$. $E = \max(12 - 9, 0) = 3$.

---

**Q6 (Medium):** Build a two-scenario example where netting reduces EE but increases ENE (counterparty exposure to bank).

> **Sketch:** Use scenarios where netting reduces positive tail but increases negative tail; compute both sides' exposures.

---

**Q7 (Medium):** Show numerically that $\max(V_1 + V_2, 0) \leq \max(V_1, 0) + \max(V_2, 0)$ for two states.

> **Sketch:** $(V_1, V_2) = (5, -4)$: LHS $= 1$, RHS $= 5$. $(V_1, V_2) = (-5, 4)$: LHS $= 0$, RHS $= 4$.

---

**Q8 (Medium):** Weekly vs daily VM on a 5-day rising MTM path. Compute max exposure under each.

> **Sketch:** Daily immediate gives $\approx 0$; weekly leaves exposure equal to MTM until first call.

---

**Q9 (Medium):** Using the "peak exposure" definition, explain how to compute 97.5% peak exposure from 10,000 simulated exposures.

> **Sketch:** Sort exposures at each time; pick the 250th highest value.

---

**Q10 (Medium):** Extend Example E by adding a 1-day lag; compute day-by-day exposures for a 4-day path.

---

**Q11 (Medium):** Construct a 3-trade example where splitting into two netting sets increases PFE but leaves EE unchanged.

---

**Q12 (Hard):** Create a wrong-way-risk example where exposure is higher in the "bad credit" state; compare unconditional EE vs conditional EE given default.

---

**Q13 (Hard):** Suppose IM is sized to a 99% 10-day move but MPOR operationally becomes 20 days. What happens to residual exposure?

---

**Q14 (Hard):** Propose a time-averaging scheme for EPE on a non-uniform time grid; discuss bias sources.

---

**Q15 (Hard):** In the Hull collateral sign convention, give an example where $C < 0$ increases exposure.

---

**Q16 (Hard):** Describe how margin call disputes could be modeled as stochastic delay; what impact on PFE?

---

**Q17 (Hard):** Using $CVA = \sum (1-R) q_i v_i$, map each term to a "risk report" quantity needed operationally.

---

**Q18 (Hard):** Explain why peak exposure and CVA naturally use different probability measures.

---

**Q19 (Hard):** A firm has 20 counterparties, each with a downgrade trigger at rating BBB. The firm is currently rated A. Model the conditional exposure if the firm is downgraded to BBB (assume all triggers fire simultaneously). How does this compare to unconditional exposure?

---

**Q20 (Hard):** Consider a CSA that permits rehypothecation. If the bank rehypothecates \$100M of collateral received from Counterparty A to Counterparty B, and then Counterparty B defaults, what is the bank's exposure to A under the Hull convention? Assume A's trades are worth +\$100M to the bank.

---

## Source Map

### (A) Verified Facts (Source-Backed)

| Fact | Source |
|------|--------|
| Exposure without collateral: $\max(V, 0)$ where $V$ is net MTM | Hull RM Ch 20 |
| Exposure with collateral: $E = \max(V - C, 0)$ | Hull RM Ch 20 (Eq. 20.4) |
| Netting treats covered trades as single transaction | Hull RM Ch 15, Ch 20 |
| Without netting: $\sum \max(V_i, 0)$ | Hull RM Ch 20 |
| Cure period/MPOR typically 10-20 days | Hull RM Ch 20 |
| Collateral at default reflects value $c$ days earlier | Hull RM Ch 20 (Example 20.1) |
| Peak exposure is high percentile of simulated exposures | Hull RM Ch 20 |
| Peak exposure conceptually uses real-world simulation | Hull RM Ch 20 |
| CVA formula: $\sum (1-R) q_i v_i$ | Hull RM Ch 20 (Eq. 20.1) |
| IM for bilateral: 10-day stressed 99% confidence | Hull RM Ch 18 |
| CSA specifies thresholds, MTAs, independent amounts | Hull RM Ch 18, Ch 20 |
| IRS exposure hump-shaped; currency swap increasing | Hull RM Ch 20 (Figure 20.1) |
| Wrong-way risk: speculating counterparties | Hull RM Ch 20 |
| Right-way risk: hedging counterparties | Hull RM Ch 20 |
| Group of Thirty: assess current and potential future exposure | Hull RM Ch 15 |
| Aggregate exposures reflecting enforceable netting | Hull RM Ch 15 |
| Rehypothecation definition | Hull RM Ch 18 |
| Lehman rehypothecation problems for hedge fund clients | Hull RM Ch 18 |
| Downgrade triggers and their risks | Hull RM Ch 20 |
| AIG downgrade trigger example (September 15, 2008) | Hull RM Ch 20 |
| Lehman had 1.5M derivatives with 8,000 counterparties | Hull RM Ch 20 (footnote 5) |

### (B) Reasoned Inference (Derived from Sources)

| Inference | Derivation |
|-----------|------------|
| EE as $\mathbb{E}[E(t)]$ | Consistent with "expected exposure" terminology in Hull |
| PFE as quantile of $E(t)$ | Matches "high percentile of simulated exposures" |
| Netting benefit inequality | Mathematical consequence of max subadditivity |
| EPE as time-average of EE | Common desk convention; sources don't fix single definition |
| CVA preview approximation | Reinterpretation of Hull's $(1-R) q_i v_i$ structure |

### (C) Flagged Uncertainties

| Topic | Uncertainty |
|-------|-------------|
| EPE/ENE exact definitions | Desks vary (time grids, discounting, collateral inclusion) |
| Settlement-lag vs MPOR | Sources don't separate intraday mechanics from MPOR |
| CSA haircuts | Not defined in primary sources for this chapter |
| Operational timing conventions | Call cutoffs, settlement cycles, dispute resolution |
