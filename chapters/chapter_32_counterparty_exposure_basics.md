# Chapter 32: Counterparty Exposure Basics — Netting, Collateral, and Margin Timing

---

## Introduction

When Lehman Brothers filed for bankruptcy on September 15, 2008, it had approximately 1.5 million derivatives transactions outstanding with about 8,000 counterparties. Each of those counterparties faced an immediate question: *How much do they owe us, and how much of that will we ever recover?*

This question—the *exposure* question—is the foundation of counterparty credit risk. Before you can price CVA, stress-test a portfolio, or set a credit limit, you must first answer: what is the potential loss if the counterparty defaults tomorrow? And critically: how does that exposure evolve over time as markets move?

The challenge is that derivatives exposure is unlike loan exposure. A loan's exposure is simply its outstanding principal. But a derivative's exposure is uncertain—it depends on market movements, the structure of the trade, and the web of legal and operational arrangements that govern what happens when things go wrong. As Hull observes in *Risk Management and Financial Institutions*, a dealer's exposure on a derivative "can be zero today and substantial tomorrow, or vice versa"—the bank might be in-the-money or out-of-the-money depending on how rates, FX, or credit spreads move.

This chapter builds the **exposure primitives**—the building blocks you need before tackling CVA, XVA, or any counterparty risk framework:

1. **Exposure as a function of MTM** — Why exposure equals $\max(V, 0)$ and how collateral modifies this
2. **Netting and close-out** — How ISDA master agreements transform counterparty risk and why "netting sets" matter
3. **Collateral mechanics** — VM, IM, thresholds, MTAs, haircuts, and how they reduce (but never eliminate) exposure
4. **Central clearing vs. bilateral clearing** — How CCPs change the exposure landscape
5. **The margin period of risk (MPOR)** — Why even "fully collateralized" trades carry residual exposure
6. **Exposure simulation** — How Monte Carlo generates PFE and EE profiles
7. **Incremental exposure** — How new trades affect portfolio CVA
8. **Wrong-way risk** — When exposure and default probability move together

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
| $v_i$ | PV of expected exposure at midpoint of interval $i$ | Currency |
| $q_i$ | Risk-neutral probability of default during interval $i$ | Dimensionless |

---

## 32.1 Exposure as a Function of MTM

### 32.1.1 The Basic Exposure Identity

Consider a dealer with a single derivatives transaction outstanding with a counterparty. The value of the transaction to the dealer is $V$. If the counterparty defaults, what does the dealer lose?

If $V < 0$, the dealer owes the counterparty—there is no credit loss (in fact, the dealer benefits from not having to pay). But if $V > 0$, the counterparty owes the dealer, and the dealer becomes an unsecured creditor for the amount $V$.

Hull makes this precise: "The dealer's net exposure is $\max(V, 0)$ where the variable $V$ is the market value of the derivative at the time of the default."

$$\boxed{E(t) = \max(V(t), 0)}$$

> **Analogy: The Casino Chip**
>
> Counterparty risk is like holding a casino chip.
>
> *   **If you are winning (\$1000 chip)**: You have exposure to the casino. You worry: "Does the cage have enough cash to pay me?"
> *   **If you are losing**: You owe the casino. You have *no* exposure to them (they have exposure to *you*).
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

> **Desk Reality: Why Exposure Metrics Matter for Capital**
>
> Banks care intensely about exposure metrics because they drive regulatory capital. Under the Basel framework, the credit equivalent amount for a derivative is:
>
> $$\boxed{\text{Credit Equivalent} = \max(V, 0) + aL}$$
>
> where $V$ is current value, $a$ is an add-on factor (from Table 15.2 in Hull RM), and $L$ is the principal amount. The add-on captures potential future exposure.
>
> Hull provides the example: "A bank has entered into a \$100 million interest rate swap with a remaining life of four years. The current value of the swap is \$2.0 million. In this case, the add-on amount is 0.5% of the principal so that the credit equivalent amount is \$2.0 million plus \$0.5 million or \$2.5 million."
>
> The capital charge is then 8% of (Credit Equivalent × Risk Weight). This is why traders care about both current exposure (for today's capital) and PFE (for limit utilization).

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

Hull emphasizes the practical importance: "Consider a bank that has three swap transactions outstanding with a particular counterparty. The transactions are worth +\$24 million, -\$17 million, and +\$8 million to the bank... Without netting, the counterparty would default on the first transaction, keep the second transaction, and default on the third transaction. Assuming no recovery, the loss to the bank would be \$32 (= 24 + 8) million. With netting, the counterparty is required to default on the second transaction as well. The loss to the bank is then \$15 (= 24 - 17 + 8) million."

### 32.2.2 The Mathematics of Netting

Let trades have individual values $V_1, V_2, \ldots, V_N$ at default time.

**Without netting** (trade-by-trade exposure):

$$E_{\text{no net}} = \sum_{i=1}^{N} \max(V_i, 0)$$

Each positive trade value contributes to exposure; negative trade values are ignored (you still owe them, but you're at risk on the positive ones).

**With netting** (net portfolio exposure):

The "single transaction" principle means the close-out is computed on the net value:

$$V_{\text{net}} = \sum_{i=1}^{N} V_i$$

$$\boxed{E_{\text{net}} = \max\left(\sum_{i=1}^{N} V_i, 0\right)}$$

Hull provides the elegant interpretation: "Without netting, the exposure is the payoff from a portfolio of options. With netting, the exposure is the payoff from an option on a portfolio."

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

### 32.3.4 Haircuts and Eligible Collateral

Not all collateral is equal. When securities (rather than cash) are posted as collateral, their value is reduced by a **haircut** to account for potential price declines before liquidation.

Hull explains in the context of exchange margins: "The percentage by which the market value of a security is reduced to determine its value for the purposes of the margin account is known as the haircut applied to the security. For example, Treasury bills might be marginable at 90% of their value (a 10% haircut), shares of a stock might be marginable at 70% of their value (a 30% haircut), and so on."

**Typical Haircut Schedule:**

| Collateral Type | Typical Haircut |
|-----------------|-----------------|
| Cash (same currency) | 0% |
| Cash (different currency) | 5-8% |
| Government bonds (AAA, <1yr) | 0.5-1% |
| Government bonds (AAA, 1-5yr) | 2-4% |
| Government bonds (AAA, >5yr) | 4-8% |
| Investment-grade corporate bonds | 8-15% |
| Equities (major indices) | 15-25% |

The effective collateral value is:

$$\boxed{C_{\text{effective}} = C_{\text{market}} \times (1 - \text{haircut})}$$

> **Desk Reality: Cheapest-to-Deliver Collateral**
>
> When a CSA permits multiple collateral types, the posting party will optimize by posting the "cheapest-to-deliver" collateral—the one with the lowest opportunity cost after haircut. If you can post either \$100 cash or \$115 of Treasuries (with a 13% haircut giving \$100 effective), you'll post whichever costs less to fund.
>
> This creates collateral optionality that affects exposure and valuation. When USD rates rise and Treasury prices fall, a counterparty might switch from posting Treasuries to posting cash. The receiving party must track this optionality.

### 32.3.5 Thresholds and MTAs

A **threshold** means VM is required only above a certain amount. Hull's example: "if value is 9 and threshold is 10, no collateral; if value is 11, collateral of 1."

The threshold creates a **residual unsecured band**—even with VM in place, exposure up to the threshold level remains uncollateralized.

The **minimum transfer amount (MTA)** avoids operational "nuisance" transfers for small amounts. If the required transfer is less than the MTA, no transfer occurs.

Both features increase residual exposure but reduce operational friction.

### 32.3.6 Rehypothecation

**Rehypothecation** refers to the practice where a party that receives collateral reuses that collateral to meet its own margin requirements elsewhere. Hull defines it as "the use of collateral posted by one counterparty to satisfy the collateral requirements of another counterparty."

The practice improves market liquidity but introduces additional risk. If the party holding your collateral defaults, you may not get that collateral back—particularly if it has been rehypothecated to third parties. Hull notes that "after Lehman Brothers declared bankruptcy in September 2008, clients (particularly European hedge fund clients) found it difficult to get a return of the collateral they had posted with Lehman because it had been rehypothecated."

As a result, post-crisis regulations and CSA amendments now commonly include clauses banning or limiting rehypothecation. Initial margin under bilateral clearing regulations must typically be segregated with a third-party custodian precisely to prevent rehypothecation.

### 32.3.7 Downgrade Triggers

CSAs may include **downgrade triggers**—clauses stating that if one party's credit rating falls below a specified level, the other party can demand additional collateral or terminate all transactions.

Hull provides the AIG example: "Many of AIG's transactions stated that AIG did not have to post collateral provided its credit rating remained above AA. However, once it was downgraded below AA, collateral was required." When AIG was downgraded by all three rating agencies on September 15, 2008, this triggered massive collateral calls that AIG could not meet, ultimately requiring a government bailout.

Downgrade triggers are a double-edged sword:
- They provide some protection against counterparty deterioration
- But they do not protect against sudden "jump-to-default" events (e.g., AA to default)
- If a company has many downgrade triggers across its contracts, a single downgrade can trigger cascade effects—precisely when the company is least able to respond

> **Practitioner Note: The Daily Margin Call Workflow**
>
> The operational workflow for margin calls typically follows this sequence:
>
> 1. **Valuation (T, morning):** Both parties independently value all trades in the netting set
> 2. **Call calculation:** Compare valuations, compute required transfer per CSA terms
> 3. **Call issuance (T, early afternoon):** Demanding party sends margin call
> 4. **Dispute window (T to T+1):** Receiving party can dispute valuation
> 5. **Settlement (T+1 or T+2):** Collateral transferred if no dispute
>
> **Common dispute causes:**
> - MTM disagreement (curve differences, model differences)
> - Trade population mismatch (one side missing a trade)
> - Incorrect collateral balance (prior transfers not reflected)
>
> **Operational implication:** Disputes effectively extend MPOR. If a counterparty is deteriorating and disputes every call for 5 days, your effective cure period may be 15+ days, not the 10 days assumed in your model.

---

## 32.4 Central Clearing vs. Bilateral Clearing

### 32.4.1 The Two Clearing Regimes

Hull explains that there are "two main approaches" to clearing OTC derivatives: bilateral clearing and central clearing.

**Bilateral clearing:** Each pair of market participants enters into an ISDA master agreement with a credit support annex (CSA) defining collateral arrangements. The parties have direct exposure to each other.

**Central clearing:** A central counterparty (CCP) interposes itself between the two parties. Hull describes: "Suppose that two companies, A and B, agree to an over-the-counter derivatives transaction. They can then present it to a CCP for clearing... The CCP acts as an intermediary and enters into offsetting transactions with the two companies." After clearing, A has exposure only to the CCP, and B has exposure only to the CCP—neither has direct exposure to the other.

### 32.4.2 How CCPs Operate

A CCP operates similarly to an exchange clearing house:

1. **Initial margin:** Both parties post IM to the CCP. Hull notes that "typically, the initial margin is calculated so that it is 99% certain to cover market moves over five days."

2. **Variation margin:** Daily (or intraday) VM flows through the CCP. "If, on the first day, interest rates fall so that the value of the swap to Company A goes down by \$100,000, Company A would be required to pay a variation margin equal to this to the CCP and the CCP would be required to pay the same amount to Company B."

3. **Default fund contributions:** Members contribute to a guaranty fund that mutualizes losses beyond a defaulting member's margin.

### 32.4.3 The CCP Default Waterfall

If a CCP member defaults, losses are absorbed in a specific order. Hull describes the **waterfall**:

1. The initial margin of the defaulting member
2. The default fund contribution of the defaulting member
3. The default fund contributions of other members
4. The equity of the CCP

This structure means that even highly creditworthy counterparties face some mutualized credit risk when clearing through a CCP—they could lose part of their default fund contribution if another member defaults.

### 32.4.4 Bilateral vs. CCP: A Comparison

| Feature | Bilateral Clearing | CCP Clearing |
|---------|-------------------|--------------|
| **Counterparty exposure** | To original counterparty | To CCP |
| **Netting** | Within bilateral netting set | Across all trades cleared at that CCP |
| **Initial margin** | Negotiated (post-crisis: mandatory for financial institutions) | Standardized, typically 5-day 99% |
| **Default handling** | Early termination under ISDA | CCP manages close-out |
| **Mutualization** | None | Default fund contributions |
| **Regulatory capital** | Higher risk weights | Lower risk weights (CCP treated as low risk) |

### 32.4.5 Netting Efficiency and Fragmentation

Central clearing can increase netting efficiency because a market participant can net all trades cleared at a single CCP, even if originally transacted with different counterparties. Hull notes this potential benefit.

However, Hull also warns about fragmentation: "There will be many CCPs and it is quite likely that they will not cooperate with each other to reduce initial margin requirements." If a bank clears interest rate swaps at one CCP and credit derivatives at another, it cannot net across CCPs—losing the netting benefit.

Moreover, some trades cannot be centrally cleared (nonstandard transactions, some FX transactions). Hull shows that "it is even possible that the new rules requiring the use of CCPs could reduce rather than increase netting in some cases" because standard trades that previously netted with nonstandard trades under bilateral agreements are now separated.

> **Desk Reality: Why Clearing Matters for Credit Limits**
>
> A trade executed bilaterally with Counterparty X consumes credit limit against X. The same trade cleared through a CCP consumes no credit limit against X—instead, it requires IM and default fund contributions to the CCP.
>
> For a trading desk, this changes deal economics:
> - Bilateral trade: Need credit approval, may hit limit, CVA charge against X
> - Cleared trade: No credit limit usage against X, but must fund IM
>
> Many desks now have "clearing mandates" requiring standard trades to be cleared whenever possible, precisely to preserve bilateral credit lines for nonstandard trades that *must* be bilateral.

---

## 32.5 The Margin Period of Risk (MPOR)

### 32.5.1 Why MPOR Matters

Even with daily variation margin and zero thresholds, exposure is not zero. The reason is timing: there is a gap between when the counterparty stops posting collateral and when trades are actually closed out.

Hull defines the **cure period** (also called margin period of risk or MPOR) as "the time between the counterparty stopping collateral posting and close-out; typically 10 or 20 days."

During this period:
1. The counterparty is in distress and stops responding to margin calls
2. The bank must recognize the default, invoke early termination
3. The bank must value the portfolio and execute close-out
4. Throughout, the market is moving—but collateral is frozen

### 32.5.2 Collateral at Default Is Stale

The key implication: **collateral at default reflects the portfolio value $c$ days earlier**, not the current value. Hull emphasizes that "the Monte Carlo simulation to calculate the $v_i$ must be structured so that the value of the derivatives portfolio with the counterparty is calculated at times $t_i^* - c$ as well as at time $t_i^*$."

This stale-collateral effect means that even under "perfect collateralization," exposure can emerge from market moves during MPOR.

> **Analogy: The Stale GPS**
>
> Collateral under MPOR is like a GPS that shows where you were 20 minutes ago, not where you are now. If you're driving fast on a twisting road, your "position" shown on the GPS (collateral) can be very different from your actual position (MTM). The faster the market moves, the larger this gap.

### 32.5.3 Hull's MPOR Example

Hull provides a concrete illustration (Example 20.1 in *Risk Management and Financial Institutions*):

> There is a two-way zero threshold collateral agreement between a bank and its counterparty. The cure period is 20 days.
>
> 1. On a particular simulation trial, the value of outstanding transactions to the bank at time $\tau$ is 50 and their value 20 days earlier is 45. The calculation assumes that the bank has collateral worth 45 in the event of a default at time $\tau$. The bank's exposure is therefore 5.
>
> 2. On a particular simulation trial, the value of outstanding transactions to the bank at time $\tau$ is 50 and their value 20 days earlier is 55. In this case, the bank will have adequate collateral and its exposure is zero.

The first scenario shows how MPOR creates exposure: the portfolio moved from 45 to 50 during the cure period, but collateral was stuck at 45.

---

## 32.6 Exposure Metrics: EE, ENE, PFE, and EPE

### 32.6.1 Expected Exposure (EE)

**Expected exposure at time $t$** is the expected value of exposure at that time:

$$\boxed{EE(t) := \mathbb{E}[E(t)] = \mathbb{E}[\max(V(t) - C(t), 0)]}$$

This is the average exposure across all possible market scenarios at time $t$. EE is the exposure measure that enters CVA calculations because CVA is an expected loss, and expected loss involves expected exposure.

### 32.6.2 Expected Negative Exposure (ENE) and DVA

While EE measures the bank's exposure to the counterparty, **expected negative exposure (ENE)** measures the counterparty's exposure to the bank:

$$\boxed{ENE(t) := \mathbb{E}[\max(-V(t), 0)]}$$

ENE is the mirror image of EE. When $V < 0$ (bank owes counterparty), the counterparty has positive exposure to the bank.

**Connection to DVA:** Just as CVA uses EE (the counterparty's expected exposure to the bank feeds into CVA calculation), **DVA (Debit Value Adjustment)** uses ENE. Hull explains: "DVA is the expected cost to the counterparty because the dealer might default. It is the counterparty's CVA."

The bilateral valuation with credit risk becomes:

$$\boxed{f^* = f_{\text{nd}} - CVA + DVA}$$

where $f_{\text{nd}}$ is the value assuming neither side will default.

> **Practitioner Note: The DVA Controversy**
>
> DVA is controversial because it represents a gain to the dealer from the possibility of its *own* default. Hull notes: "One surprising effect of DVA is that when the credit spread of a derivatives dealer increases, DVA increases. This in turn leads to an increase in the reported value of the derivatives on the books of the dealer and a corresponding increase in its profits."
>
> Regulators are uncomfortable with this, and "have excluded DVA gains and losses from the definition of common equity in the determination of regulatory capital."

### 32.6.3 Potential Future Exposure (PFE)

**Potential future exposure** is a quantile of the exposure distribution:

$$\boxed{PFE_q(t) := \inf\{x : P(E(t) \leq x) \geq q\}}$$

The $q$-quantile (e.g., 95% or 97.5%) answers: "What exposure level will we not exceed with probability $q$?"

Hull describes peak exposure as "a high percentile of simulated exposures at each future time." For example, with 10,000 Monte Carlo trials and a 97.5% percentile, the peak exposure at time $t$ is the 250th highest exposure recorded at that time.

**PFE vs. EE:** PFE answers "how bad can it get?" while EE answers "what's the average?" PFE is used for credit limits and stress testing; EE feeds into CVA pricing.

### 32.6.4 Expected Positive Exposure (EPE)

**EPE** is a time-aggregated measure of expected exposure. The common convention is a time-weighted average:

$$EPE := \frac{1}{T} \int_0^T EE(t) \, dt \quad \text{or discretized:} \quad EPE \approx \sum_i w_i \, EE(t_i)$$

EPE provides a single-number summary of the exposure profile over the life of the portfolio. It is used in regulatory capital calculations and as an input to certain CVA approximations.

> **Note on uncertainty:** The precise definition of EPE (time-weighting, discounting, whether collateral is included) varies across desks and regulatory frameworks. To be certain, verify your desk's specific definition.

### 32.6.5 Risk-Neutral vs. Real-World Simulation

Hull flags an important conceptual distinction: CVA is a valuation exercise requiring risk-neutral simulation, but PFE is scenario analysis asking "how bad can it get?"

"When we calculate peak exposure, we are carrying out a scenario analysis... For this purpose, we should in theory simulate the behavior of market variables in the real world, not the risk-neutral world."

In practice, this distinction is often ignored, and the same risk-neutral simulations are used for both purposes. But understanding the conceptual difference matters for interpreting results.

---

## 32.7 Monte Carlo Simulation for Exposure

### 32.7.1 The Simulation Framework

Computing exposure metrics (EE, PFE, EPE) requires simulating how portfolio values evolve under many possible future scenarios. This section explains the Monte Carlo procedure.

**Step 1: Identify Risk Factors**

The portfolio value $V(t)$ depends on market variables: interest rates, FX rates, credit spreads, equity prices, volatilities. These are the **risk factors** to be simulated.

For an interest rate swap portfolio, typical risk factors include:
- Short rate or yield curve level
- Curve slope and curvature
- Volatilities (if options are present)

**Step 2: Generate Scenarios**

For each simulation path $\omega$ and each future time $t_i$:
1. Simulate market variable evolution: $X^{(\omega)}(t_i)$
2. These may use geometric Brownian motion, short-rate models, or more sophisticated processes

**Step 3: Compute Portfolio Values**

At each future time $t_i$ on each path $\omega$:
1. Price all trades in the netting set using the simulated market state
2. Sum to get $V^{(\omega)}(t_i)$

**Step 4: Apply Collateral Model**

If the portfolio is collateralized:
1. Compute collateral $C^{(\omega)}(t_i)$ based on the CSA rules
2. Account for MPOR: $C^{(\omega)}(t_i)$ reflects $V^{(\omega)}(t_i - c)$, not $V^{(\omega)}(t_i)$

**Step 5: Compute Exposure**

$$E^{(\omega)}(t_i) = \max(V^{(\omega)}(t_i) - C^{(\omega)}(t_i), 0)$$

**Step 6: Aggregate Across Paths**

$$EE(t_i) = \frac{1}{N} \sum_{\omega=1}^{N} E^{(\omega)}(t_i)$$

$$PFE_{97.5\%}(t_i) = \text{97.5th percentile of } \{E^{(\omega)}(t_i)\}_{\omega=1}^{N}$$

### 32.7.2 Time Grid Selection

The choice of time grid $\{t_1, t_2, \ldots, t_n\}$ affects accuracy and computation:

- **Finer at the front end:** Exposure changes rapidly in early periods; use daily or weekly steps for the first year
- **Coarser at the back end:** For long-dated portfolios, quarterly or semi-annual steps may suffice
- **Align with cashflow dates:** Include coupon and reset dates to capture discrete jumps

> **Desk Reality: What the Quant Team Actually Does**
>
> A typical CVA simulation for a large netting set might involve:
> - 10,000 Monte Carlo paths
> - 100+ time steps over a 30-year horizon (finer at front, coarser at back)
> - 5-20 risk factors (depending on product complexity)
> - Full revaluation of hundreds of trades at each time step on each path
>
> This is computationally intensive. Many desks use approximations:
> - **American Monte Carlo (regression):** Approximate future values rather than full repricing
> - **Bucketing:** Group similar trades and compute representative exposure
> - **Adjoint Algorithmic Differentiation (AAD):** Efficiently compute sensitivities
>
> The simulation paths are often **stored** so that incremental CVA can be computed quickly when new trades are added (see Section 32.10).

### 32.7.3 The Exposure Cone

The distribution of exposure over time can be visualized as an **exposure cone**: a fan of percentile bands showing how exposure uncertainty grows (and eventually shrinks) over the portfolio's life.

For a typical interest rate swap:
- At $t = 0$: Exposure is zero (or small) since the swap is at-the-money
- At intermediate times: Exposure distribution widens as rates can move substantially
- Near maturity: Exposure shrinks because few payments remain

The 97.5% PFE line forms the "upper boundary" of the cone; the EE is the center of mass.

---

## 32.8 Exposure Profiles for Different Instruments

### 32.8.1 Interest Rate Swaps vs. Currency Swaps

Hull provides a striking comparison (Figure 20.1 in *Risk Management and Financial Institutions*):

> "The expected exposure on the interest rate swaps starts at zero, increases, and then decreases. By contrast, expected exposure on the currency swaps increases steadily with the passage of time."

Why the difference? For interest rate swaps, early in the trade there's much to be exchanged (many future cashflows), so exposure can grow as rates move. But toward the end, very little remains to be exchanged, so exposure declines. The profile is "hump-shaped."

For currency swaps, principals are exchanged at the end of the life. The uncertainty about the exchange rate at maturity drives exposure higher as maturity approaches. The profile is monotonically increasing.

This has practical implications: "The impact of default risk for a dealer in currency swaps is therefore much greater than for a dealer in interest rate swaps."

> **Visual: The Exposure Shape**
>
> *   **Interest Rate Swap ("The Hump")**: Starts low (par), grows as rates move, then falls to zero as payments are made and time runs out. Peak risk is in the middle.
> *   **FX Swap / Cross-Currency ("The Ramp")**: Starts low, but grows steadily because the huge principal exchange at the end gets riskier and riskier as FX drifts further from the strike. Peak risk is at the very end.

### 32.8.2 Forward Contracts

For a forward contract to buy an asset at price $K$ with current forward price $F_0$, Hull derives the present value of expected exposure:

$$v_i = e^{-rT}\left[F_0 N(d_1) - K N(d_2)\right]$$

where $d_1$ and $d_2$ are Black-Scholes-type quantities. The exposure looks like a call option on the forward price—which makes sense, because the bank's positive exposure occurs when the forward price exceeds the strike.

### 32.8.3 Options: Bounded and Asymmetric Exposure

Options have distinct exposure profiles depending on whether you bought or sold them.

**Bought options (long):** When you buy an option, you pay the premium upfront. Your future exposure is limited to the replacement cost of the option—at most the current market value, which is bounded by the premium paid (for capped premium structures) or can grow but is always positive.

Hull's framework shows that for a derivative "that is bound to have a positive value to the dealer... at all future times" (like a purchased option), the CVA simplifies because the exposure is simply the option value at each time.

**Sold options (short):** When you sell an option, you receive premium but have a contingent liability. From the bank's perspective, the sold option has *negative* value—so the bank has *no exposure* on a sold option. Instead, the *counterparty* has exposure to the bank.

> **Practitioner Note: Option Exposure Dynamics**
>
> For a purchased call option:
> - Exposure = option value = intrinsic + time value
> - As expiry approaches, time value decays (Theta)
> - But if in-the-money, intrinsic value grows with the underlying
>
> This creates a complex exposure profile. For at-the-money options, exposure typically follows a "decay then spike" pattern: initially decaying as time value erodes, but potentially spiking if the option goes deep in-the-money near expiry.
>
> For swaptions and other rate options, the profile depends on whether strike rates are hit. A receiver swaption gains value as rates fall—its exposure profile looks like a call option on rates.

---

## 32.9 Wrong-Way Risk (Preview)

### 32.9.1 Definition and Intuition

**Wrong-way risk** occurs when exposure and the probability of default are positively correlated. Hull defines it as the situation where "exposure rises when counterparty becomes more likely to default."

The intuition is captured by the phrase: "The counterparty is weakest exactly when I'm most exposed."

### 32.9.2 Examples of Wrong-Way Risk

Hull provides several examples:

**Speculating counterparty:** "A situation in which a company is speculating by entering into many similar trades with one or more dealers is likely to lead to wrong-way risk for these dealers. This is because the company's financial position and therefore its probability of default is likely to be affected adversely if the trades move against the company."

**CDS seller:** A counterparty selling credit protection on a reference entity may experience wrong-way risk for the protection buyer. If the reference entity defaults, the protection becomes very valuable—exactly when the protection seller (who has similar credit characteristics) may also be in distress.

> **Classic Example: The Put Option on the Bank**
>
> If you buy a put option on Bank A's stock *from* Bank A...
> *   **The Payoff**: You make money if Bank A's stock crashes.
> *   **The Catch**: If Bank A's stock crashes, Bank A might be bankrupt.
> *   **Result**: You are "winning" exactly when the person paying you is "dying." Your expected recovery is terrible. This is extreme wrong-way risk.

### 32.9.3 Right-Way Risk

The opposite is **right-way risk**: exposure and default probability are negatively correlated.

Hull notes: "If a company enters into transactions with a dealer to partially hedge an existing exposure, there should in theory be right-way risk. This is because, when the transactions move against the counterparty, it will be benefiting from the unhedged portion of its exposure so that its probability of default should be relatively low."

### 32.9.4 Modeling Wrong-Way Risk

Hull describes the regulatory approach: "A simple way of dealing with wrong-way risk is to use what is termed the 'alpha' multiplier to increase $v_i$... Basel II rules set alpha equal to 1.4, but allow banks to use their own models, with a floor for alpha of 1.2."

This means CVA calculated under the independence assumption must be scaled up by at least 20% (alpha = 1.2) to account for potential wrong-way risk.

More sophisticated approaches model the dependence directly. Hull references research where "the hazard rate at time $t$ is a function of variables observable at that time," allowing exposure and credit quality to co-move.

> **Scope note:** Quantitative modeling of wrong-way risk requires specifying the dependence between market variables and default probability. This is a separate (and difficult) modeling problem, treated in the XVA chapters. Here we only preview the concept.

---

## 32.10 Incremental Exposure and CVA

### 32.10.1 Why Incremental CVA Matters

When a trader considers adding a new trade to an existing portfolio with a counterparty, the key question is: *How does this trade affect the CVA we charge?*

The answer is **not** simply the standalone CVA of the new trade. Because of netting, the new trade interacts with existing trades:
- If the new trade has *opposite* exposure to the existing portfolio, it provides a netting benefit and *reduces* total exposure
- If the new trade has *similar* exposure to the existing portfolio, it adds to exposure

### 32.10.2 Incremental vs. Standalone

Hull explains the distinction: "If the value of the new transaction is positively correlated with other transactions... the incremental effect on CVA is likely to be positive. If it is negatively correlated, the effect could well be negative."

$$\boxed{\Delta CVA = CVA_{\text{portfolio + new}} - CVA_{\text{portfolio}}}$$

This **incremental CVA** can be negative—meaning the new trade *reduces* the credit charge because of netting benefits.

### 32.10.3 Efficient Computation

Hull notes that "calculating the incremental effect of a new transaction on CVA by recomputing CVA is not usually feasible" because full CVA calculation is computationally intensive.

The efficient approach: **store simulation paths** from the original CVA calculation. When a new trade is proposed:
1. Price the new trade on each stored path at each time step
2. Add to the portfolio values already stored
3. Recompute exposure on each path with the combined portfolio
4. Calculate the new CVA and compare

This "incremental CVA engine" allows traders to get real-time quotes on the credit charge for new trades.

### 32.10.4 Pricing Implications

> **Desk Reality: Pricing a New Trade**
>
> When a salesperson asks for a price on a new trade with an existing counterparty:
>
> 1. **Compute standalone price** (as if no credit risk)
> 2. **Query incremental CVA** from the CVA desk
> 3. **Adjust bid/offer** to include CVA
>
> If the new trade *reduces* portfolio exposure (e.g., an offsetting swap), the incremental CVA may be negative—the credit desk effectively "pays" the trading desk for the netting benefit. This can make otherwise marginal trades attractive.
>
> If the new trade *increases* exposure (e.g., same-direction risk), the CVA charge widens the bid-offer, making the trade less competitive.

---

## 32.11 CVA Preview: How Exposure Metrics Feed Into Valuation

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

## 32.12 Worked Examples

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

### Example K: Incremental Exposure (Netting Benefit)

**Setup:** Existing portfolio has $V_{\text{existing}} = +10$ with certainty. New trade has $V_{\text{new}} = -6$ with certainty (perfectly offsetting direction).

**Before new trade:**
$$E_{\text{before}} = \max(10, 0) = 10$$

**After new trade (with netting):**
$$V_{\text{combined}} = 10 - 6 = 4$$
$$E_{\text{after}} = \max(4, 0) = 4$$

**Incremental exposure:**
$$\Delta E = E_{\text{after}} - E_{\text{before}} = 4 - 10 = -6$$

**Interpretation:** The new trade *reduces* exposure by 6 due to netting. The incremental CVA would be negative—the credit desk "pays" for the benefit.

---

### Example L: CVA Preview Calculation

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

## 32.13 Practical Notes

### 32.13.1 Common Pitfalls

**Computing exposure trade-by-trade instead of at netting set level:** This violates netting logic and overstates exposure. Always aggregate within enforceable netting agreements.

**Forgetting timing lags and MPOR:** Collateral is stale by $c$ days. Exposure can appear even under perfect collateralization if MTM moves during the cure period.

**Confusing PFE with EE:** PFE (quantile) answers "how bad?"; EE (mean) answers "on average." Use the right measure for the right purpose—PFE for limits, EE for CVA.

**Sign convention errors:** Always verify that positive $V$ means in-the-money for the bank, and that collateral $C$ follows the correct convention. Sign errors reverse conclusions.

**Ignoring haircuts on securities collateral:** \$100 of Treasury bonds is not \$100 of collateral. After a 5% haircut, it's \$95 effective.

### 32.13.2 Implementation Considerations

**Time grid choices:** Exposure profiles depend on discretization. Finer grids near the front end (where exposure changes rapidly) may be needed.

**Collateral modeling:** Frequency, thresholds, MTAs, haircuts, and "stale collateral" assumptions can dominate exposure outcomes. Model these carefully.

**Simulation requirements:** Hull notes that dealers may have transactions with thousands of counterparties, making CVA calculation computationally intensive. Efficient simulation and storage of paths is essential.

**CCP vs. bilateral treatment:** Ensure your systems correctly identify which trades are centrally cleared (exposure to CCP) vs. bilateral (exposure to counterparty).

### 32.13.3 Sanity Checks

- **Netting cannot increase exposure:** If $E_{\text{net}} > E_{\text{no net}}$ in your model, something is wrong.
- **Tighter collateral reduces exposure:** Lower thresholds, lower MTAs, and more frequent VM should all reduce exposure metrics.
- **PFE ≥ EE** for typical distributions. Equality occurs only for degenerate cases (constant exposure).
- **IRS exposure is hump-shaped; FX swap exposure increases:** If your model shows opposite shapes, investigate.
- **Incremental exposure can be negative:** A trade that offsets existing exposure has negative incremental exposure and CVA.

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

8. **ENE feeds DVA:** Expected negative exposure is the counterparty's view, used for DVA.

9. **Exposure profiles differ by product:** Interest rate swaps have hump-shaped profiles; currency swaps increase monotonically; bought options are bounded.

10. **Central clearing changes exposure:** Exposure is to the CCP, not the original counterparty; netting is across all trades at that CCP.

11. **Incremental CVA can be negative:** New trades that reduce portfolio exposure provide a netting benefit.

12. **Wrong-way risk is dangerous:** Exposure rising exactly when default probability rises amplifies losses.

13. **CVA uses exposure primitives:** $CVA = \sum (1-R) q_i v_i$ where $v_i$ comes from exposure modeling.

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **MTM** $V(t)$ | Net mark-to-market value of netting set | The input to exposure calculation |
| **Exposure** $E(t)$ | $\max(V-C, 0)$—positive part of (MTM minus collateral) | What you lose at counterparty default |
| **EE** | $\mathbb{E}[E(t)]$—expected exposure | Feeds into CVA calculation |
| **ENE** | $\mathbb{E}[\max(-V,0)]$—expected negative exposure | Feeds into DVA calculation |
| **PFE** | Quantile of $E(t)$—"how bad can it get?" | Used for credit limits |
| **EPE** | Time-average of EE | Single-number exposure summary |
| **Netting** | Legal aggregation under master agreement | Reduces exposure via offset |
| **VM** | Collateral tracking current MTM | Reduces current exposure |
| **IM** | Buffer for MPOR period | Covers 10-day stressed moves |
| **MPOR** | Cure period (10-20 days) | Makes collateral stale |
| **Threshold** | Level below which no VM required | Creates residual unsecured band |
| **MTA** | Minimum transfer amount | Avoids nuisance transfers |
| **Haircut** | Reduction in collateral value for securities | Adjusts effective collateral |
| **CCP** | Central counterparty | Changes exposure counterparty, increases netting |
| **Default fund** | CCP member contribution to mutualize losses | Shared loss after IM exhausted |
| **Rehypothecation** | Reuse of received collateral for other purposes | Creates recovery risk if holder defaults |
| **Downgrade trigger** | CSA clause demanding collateral on rating downgrade | Protection vs. cascade risk |
| **Wrong-way risk** | Exposure ↑ when default probability ↑ | Amplifies losses |
| **Incremental CVA** | CVA change from adding a new trade | Can be negative (netting benefit) |

---

## Notation for This Chapter

| Symbol | Definition |
|--------|------------|
| $V(t)$ | Net MTM of netting set at time $t$ |
| $C(t)$ | Collateral held by bank (positive = received) |
| $E(t)$ | Bank's exposure: $\max(V-C, 0)$ |
| $EE(t)$ | Expected exposure at time $t$ |
| $ENE(t)$ | Expected negative exposure at time $t$ |
| $PFE_q(t)$ | $q$-quantile of exposure at time $t$ |
| $EPE$ | Expected positive exposure (time-average of EE) |
| $H$ | Threshold |
| $m$ | Minimum transfer amount (MTA) |
| $c$ | Cure period / MPOR (days) |
| $v_i$ | PV of expected exposure at midpoint of interval $i$ |
| $q_i$ | Default probability during interval $i$ |
| $R$ | Recovery rate |

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
| 7 | Define EE(t). | $EE(t) = \mathbb{E}[E(t)]$—expected exposure at time $t$. |
| 8 | Define ENE(t). | $ENE(t) = \mathbb{E}[\max(-V(t), 0)]$—expected negative exposure, used for DVA. |
| 9 | Define PFE. | The $q$-quantile of $E(t)$—e.g., 95th percentile of exposure. |
| 10 | What is variation margin (VM)? | Collateral posted to track positive MTM to the other side. |
| 11 | What is initial margin (IM) designed to cover? | Potential exposure during the margin period of risk (10-day stressed moves at 99% confidence). |
| 12 | What is the cure period / MPOR? | Delay (typically 10-20 days) between collateral cessation and close-out. |
| 13 | Why does MPOR matter? | Collateral at default reflects value $c$ days earlier, creating exposure if MTM moves. |
| 14 | What is a haircut? | Reduction applied to securities collateral to account for potential price decline. |
| 15 | What is a threshold in a CSA? | Level below which no VM is required—creates residual unsecured exposure. |
| 16 | What is MTA? | Minimum transfer amount—avoids small collateral movements. |
| 17 | What is a CCP? | Central counterparty that interposes itself between trade parties, becoming counterparty to both. |
| 18 | What is the CCP default waterfall? | IM of defaulter → Default fund of defaulter → Default fund of other members → CCP equity. |
| 19 | How does central clearing change netting? | Trades with different counterparties can be netted if all cleared at the same CCP. |
| 20 | What is wrong-way risk? | Exposure rising when counterparty default probability rises. |
| 21 | What is right-way risk? | Exposure falling when counterparty default probability rises. |
| 22 | What is rehypothecation? | Reuse of received collateral to meet other collateral demands. |
| 23 | How do IRS exposure profiles behave? | Hump-shaped: start at zero, increase, then decrease toward maturity. |
| 24 | How do currency swap exposure profiles behave? | Monotonically increasing (principal exchanged at end). |
| 25 | What is incremental CVA? | CVA change from adding a new trade: $\Delta CVA = CVA_{\text{new portfolio}} - CVA_{\text{old portfolio}}$. |
| 26 | Can incremental CVA be negative? | Yes—if the new trade reduces portfolio exposure through netting. |
| 27 | What is the CVA formula structure? | $CVA = \sum (1-R) q_i v_i$ |
| 28 | What measure is used for CVA? | Risk-neutral (for pricing). |
| 29 | What measure is conceptually appropriate for PFE? | Real-world (for scenario analysis), though risk-neutral often used in practice. |
| 30 | What happened to AIG due to downgrade triggers? | Downgrade below AA triggered massive collateral calls, requiring government bailout. |

---

## Mini Problem Set

**Instructions:** Problems increase in difficulty. Solution sketches provided for Q1-Q10.

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

**Q6 (Medium):** A bank posts \$110 of Treasury bonds as collateral. The haircut is 10%. The bank owes the counterparty \$95. If the counterparty defaults, what is the bank's exposure?

> **Sketch:** Effective collateral posted = $110 \times (1 - 0.10) = 99$. Bank's exposure = $\max(-95 - (-99), 0) = \max(4, 0) = 4$.

---

**Q7 (Medium):** Show numerically that $\max(V_1 + V_2, 0) \leq \max(V_1, 0) + \max(V_2, 0)$ for two states.

> **Sketch:** $(V_1, V_2) = (5, -4)$: LHS $= 1$, RHS $= 5$. $(V_1, V_2) = (-5, 4)$: LHS $= 0$, RHS $= 4$.

---

**Q8 (Medium):** A bank has trades worth +\$50mm with Counterparty X cleared bilaterally, and trades worth -\$30mm with the same counterparty cleared at CCP Alpha. Can these be netted? What is total exposure?

> **Sketch:** No—different netting sets. Bilateral exposure = $\max(50, 0) = 50$mm. CCP exposure = $\max(-30, 0) = 0$. Total = \$50mm.

---

**Q9 (Medium):** Using the "peak exposure" definition, explain how to compute 97.5% peak exposure from 10,000 simulated exposures at time $t$.

> **Sketch:** Sort the 10,000 exposures at time $t$ from lowest to highest. The 97.5th percentile is the 250th highest value (since $10,000 \times 0.025 = 250$).

---

**Q10 (Medium):** Existing portfolio has EE = 10 at all times. A new trade has standalone EE = 5. If the new trade is perfectly negatively correlated with the existing portfolio, what is the approximate EE of the combined portfolio?

> **Sketch:** If perfectly negatively correlated, the new trade offsets the existing one. Combined EE ≈ $|10 - 5| = 5$, not $10 + 5 = 15$. Incremental EE is -5 (a reduction).

---

**Q11 (Medium):** Construct a 3-trade example where splitting into two netting sets increases PFE but leaves EE unchanged.

---

**Q12 (Hard):** Create a wrong-way-risk example where exposure is higher in the "bad credit" state; compare unconditional EE vs conditional EE given default.

---

**Q13 (Hard):** Suppose IM is sized to a 99% 10-day move but MPOR operationally becomes 20 days due to disputes. What happens to residual exposure?

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

**Q20 (Hard):** Compare total exposure under: (a) all trades bilateral with 3 counterparties, (b) all trades cleared at one CCP, (c) trades split across 2 CCPs. Use a simple 6-trade example.

---

## Source Map

### (A) Book-Verified Facts

| Fact | Source |
|------|--------|
| Exposure without collateral: $\max(V, 0)$ where $V$ is net MTM | Hull RM Ch 20 |
| Exposure with collateral: $E = \max(V - C, 0)$ | Hull RM Ch 20 (Eq. 20.4) |
| Netting treats covered trades as single transaction | Hull RM Ch 15, Ch 20 |
| Without netting: $\sum \max(V_i, 0)$; with netting: option on portfolio vs. portfolio of options | Hull RM Ch 15, Ch 20 |
| Cure period/MPOR typically 10-20 days | Hull RM Ch 20 |
| Collateral at default reflects value $c$ days earlier | Hull RM Ch 20 (Example 20.1) |
| Peak exposure is high percentile of simulated exposures | Hull RM Ch 20 |
| Peak exposure conceptually uses real-world simulation | Hull RM Ch 20 |
| CVA formula: $\sum (1-R) q_i v_i$ | Hull RM Ch 20 (Eq. 20.1) |
| IM for bilateral: 10-day stressed 99% confidence | Hull RM Ch 18 |
| CCP operates like exchange clearing house | Hull RM Ch 18 |
| CCP default waterfall: IM → default fund → other members → equity | Hull RM Ch 18 |
| Central clearing increases netting across counterparties | Hull RM Ch 18 |
| Multiple CCPs reduce netting benefit (fragmentation) | Hull RM Ch 18 |
| CSA specifies thresholds, MTAs, independent amounts | Hull RM Ch 18, Ch 20 |
| Haircut definition: percentage reduction in securities value | Hull RM Ch 18 |
| IRS exposure hump-shaped; currency swap increasing | Hull RM Ch 20 (Figure 20.1) |
| Forward contract exposure formula | Hull RM Ch 20 (Eq. 20.10) |
| Wrong-way risk: speculating counterparties | Hull RM Ch 20 |
| Right-way risk: hedging counterparties | Hull RM Ch 20 |
| Alpha multiplier for wrong-way risk (Basel: 1.4, floor 1.2) | Hull RM Ch 20 |
| Group of Thirty: assess current and potential future exposure | Hull RM Ch 15 |
| Aggregate exposures reflecting enforceable netting | Hull RM Ch 15 |
| Basel credit equivalent = max(V,0) + add-on factor × L | Hull RM Ch 15 (Eq. 15.1) |
| Add-on factors by product type and maturity | Hull RM Ch 15 (Table 15.2) |
| DVA is counterparty's CVA; bilateral value = $f_{nd} - CVA + DVA$ | Hull RM Ch 20.6 |
| DVA increases when dealer's credit spread increases | Hull RM Ch 20 |
| Incremental CVA: new trade effect depends on correlation with existing | Hull RM Ch 20.3 |
| Efficient incremental CVA: store simulation paths | Hull RM Ch 20.3 |
| Rehypothecation definition | Hull RM Ch 18 |
| Lehman rehypothecation problems for hedge fund clients | Hull RM Ch 18 |
| Downgrade triggers and their risks | Hull RM Ch 20 |
| AIG downgrade trigger example (September 15, 2008) | Hull RM Ch 20 |
| Lehman had 1.5M derivatives with 8,000 counterparties | Hull RM Ch 20 (footnote 5) |

### (B) Claude-Extended Content

| Content | Basis |
|---------|-------|
| "Desk Reality: Why Exposure Metrics Matter for Capital" box | Extends Hull RM Ch 15 Basel content |
| "Desk Reality: Cheapest-to-Deliver Collateral" box | Extends Hull RM Ch 18 haircut discussion |
| "Practitioner Note: The Daily Margin Call Workflow" box | Operational extension of collateral mechanics |
| "Desk Reality: Why Clearing Matters for Credit Limits" box | Extends Hull RM Ch 18 CCP discussion |
| "Desk Reality: What the Quant Team Actually Does" box | Extends Hull RM Ch 20 simulation discussion |
| "Analogy: The Stale GPS" for MPOR | Pedagogical extension |
| Typical haircut schedule table | Standard market practice, not verbatim from Hull |
| Option exposure profile discussion | Logical extension of Hull's derivative-positive-value example |

### (C) Reasoned Inference

| Inference | Derivation |
|-----------|------------|
| EE as $\mathbb{E}[E(t)]$ | Consistent with "expected exposure" terminology in Hull |
| ENE as $\mathbb{E}[\max(-V, 0)]$ | Mirror of EE for counterparty side; consistent with DVA definition |
| PFE as quantile of $E(t)$ | Matches "high percentile of simulated exposures" |
| Netting benefit inequality | Mathematical consequence of max subadditivity |
| EPE as time-average of EE | Common desk convention; sources don't fix single definition |
| CVA preview approximation | Reinterpretation of Hull's $(1-R) q_i v_i$ structure |
| Incremental exposure formula: $\Delta E = E_{\text{combined}} - E_{\text{existing}}$ | Direct consequence of netting mechanics |
| Effective collateral formula with haircut | Direct application of haircut definition |

### (D) Flagged Uncertainties

| Topic | Uncertainty |
|-------|-------------|
| EPE/ENE exact definitions | Desks vary (time grids, discounting, collateral inclusion) |
| Settlement-lag vs MPOR | Sources don't separate intraday mechanics from MPOR |
| Specific haircut percentages | Vary by institution and CSA; table shows typical values only |
| Operational margin call timing | Call cutoffs, settlement cycles, dispute resolution procedures vary by institution |
| Wrong-way risk modeling details | Hull provides conceptual framework; specific implementation varies |
