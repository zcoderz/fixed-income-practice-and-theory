# Chapter 33: Collateral Discounting and OIS

---

## Introduction

In 2007, a swap desk priced a 10-year interest rate swap by discounting all cashflows at LIBOR. By 2010, that same desk was discounting at OIS—and the difference in present value for a large swap book could be tens of millions of dollars. What changed?

The answer lies in a seemingly mundane detail: **what rate accrues on the cash collateral that counterparties post to each other?** Before the financial crisis, the spread between overnight rates (like the Fed funds rate) and term interbank rates (like 3-month LIBOR) was small enough that desks could ignore it. When that spread blew out to 275 basis points in late 2008, the discounting basis became a material P&L driver. As Andersen and Piterbarg explain in *Interest Rate Modeling*, "it is now generally accepted that the Libor rate is no longer a good proxy for a discounting rate on collateralized trades."

This chapter explains **why collateral terms determine the discount rate** for derivative pricing. The core insight is elegant: if you post cash collateral and earn the overnight rate on it, then that overnight rate is the relevant funding rate for your hedging strategy—and hence the appropriate discount rate for the derivative. This is not a regulatory mandate or accounting quirk; it is a no-arbitrage requirement.

We cover:

1. **What collateral is and how it works** — CSA terms, variation margin, thresholds, rehypothecation, and the margin period of risk
2. **The economic principle** — why discounting follows the collateral remuneration rate, with an arbitrage-based derivation
3. **From CSA to OIS curve** — how overnight rates like Fed funds/SOFR link to the OIS discount curve
4. **Multi-curve pricing** — separating discount curves from projection curves
5. **Multi-currency CSAs** — cheapest-to-deliver collateral and how to value the optionality
6. **Imperfect collateralization** — what breaks when thresholds, lags, or defaults enter, including downgrade triggers
7. **Price Alignment Interest (PAI)** — the daily mechanics of collateral remuneration

The stakes are high: every dealer, risk manager, and quantitative analyst working with OTC derivatives must understand this material. Get the discounting wrong, and your P&L is fiction.

---

## 33.1 Collateral and CSA Mechanics

### 33.1.1 The Credit Support Annex (CSA)

A bilateral OTC derivatives relationship is typically governed by an ISDA Master Agreement with an annex called the **Credit Support Annex (CSA)** that specifies collateral requirements. Hull describes this framework in *Risk Management and Financial Institutions*: "A collateral agreement between two sides is in the credit support annex (CSA) of their ISDA master agreement. This specifies thresholds, independent amounts, minimum transfer amounts, and haircuts on noncash collateral for the two sides."

The CSA is effectively part of the product: it changes the economics of the derivative by altering how counterparty risk is managed and, critically for this chapter, by determining what discount rate should be used. A derivative with a zero-threshold two-way CSA is a fundamentally different economic instrument from the same derivative with no CSA—even if the contractual cashflows are identical.

### 33.1.2 Variation Margin (VM)

**Variation margin** is collateral exchanged as mark-to-market (MTM) changes. In a simple two-way CSA with zero threshold, if the MTM increases in value to party A by $X from one day to the next, party B posts collateral worth $X to party A (and vice versa if MTM moves the other way).

The purpose of VM is to keep current credit exposure close to zero by "resetting" collateral to match the MTM. If VM posting were continuous and frictionless, the exposure between counterparties would always be approximately zero—there would be no unsecured credit risk in the derivative.

Hull emphasizes that collateral "mitigates credit risk" because "many derivatives transactions require posting of collateral (normally cash or Treasury bonds) in the amount of the current mark-to-market." By 2009, according to ISDA data cited in *Interest Rate Modeling*, "about 65% of all OTC derivatives transactions involved such agreements."

### 33.1.3 Initial Margin (IM)

**Initial margin** is collateral posted at trade inception (or held throughout the trade's life) to protect against losses that might occur during the time it takes to close out a position after a counterparty default. While this chapter focuses on how collateral determines discounting—primarily a VM story—IM has become increasingly important under post-crisis regulations.

The distinction matters: VM aims to eliminate current exposure by matching collateral to MTM, while IM aims to cover **potential future exposure** over a liquidation period. A full treatment of IM requirements and their capital implications belongs in later chapters on regulatory capital; here we note only that IM posting has become more common due to regulation.

### 33.1.4 Thresholds and Minimum Transfer Amounts

Real-world CSAs rarely require collateral posting for every dollar of MTM movement. Two parameters create friction:

**Threshold ($H$):** The MTM level below which VM need not be posted. If the threshold is $10 million, no collateral is required when $|V| < \$10\text{m}$, but if $V = \$11\text{m}$, the counterparty must post $\$1\text{m}$ in collateral.

**Minimum transfer amount (MTA):** The smallest collateral transfer that will actually occur. If MTA is $500{,}000$ and the required collateral movement is $300{,}000$, no transfer happens. This avoids the operational cost of nuisance transfers.

Both parameters leave **residual unsecured exposure**. A threshold of $H$ means that up to $H$ in MTM is never collateralized—this exposure is economically equivalent to unsecured credit risk.

### 33.1.5 The Cure Period (Margin Period of Risk)

Even with zero threshold and daily margining, exposure is not eliminated. Hull's Example 20.1 in *Risk Management and Financial Institutions* illustrates why:

> "It is usually assumed that a period of time elapses between the time when a counterparty stops posting collateral and the close out of transactions. This period of time is known as the cure period (or margin period of risk). It is typically 10 or 20 days."

The effect is that "the collateral at the time of a default does not reflect the value of the portfolio at the time of the default. It reflects the value 10 or 20 days earlier."

**Example (from Hull):** With a two-way zero threshold collateral agreement and a 20-day cure period:
- If the portfolio value to the bank is $50 million at the midpoint of a default interval and was $45 million 20 days earlier, the bank has collateral worth $45 million but exposure of $50 million. The bank's net exposure is $5 million.
- If the portfolio value is $50 million now but was $55 million 20 days earlier, the bank has adequate collateral and exposure is zero.

This cure-period effect explains why even "fully collateralized" trades carry credit exposure and therefore require CVA (see Chapter 34).

### 33.1.6 The Collateral Remuneration Rate

The collateral remuneration rate $c(t)$ is the contractual rate paid on posted cash collateral. This rate is the central character in our discounting story.

Andersen and Piterbarg state in *Interest Rate Modeling* that in USD, the effective Federal funds rate "is often considered the best available proxy for a risk-free USD rate, in part because the Fed funds rate used to be closely linked to the overnight Libor rate... However, in the subprime crisis of 2007-2009 the two have diverged significantly." More directly, the Fed funds rate is "normally the contractual rate used to accrue interest on posted collateral."

Similar overnight rates exist for other currencies:
- **EUR:** EONIA (replaced by €STR)
- **GBP:** SONIA

These rates are "computed as averages of all actual overnight lending/borrowing transactions by qualifying banks weighted by the size of the transactions," as *Interest Rate Modeling* explains. They reflect actual transactions, in contrast to LIBOR which reflects banks' estimates of rates at which borrowing might take place.

### 33.1.7 Rehypothecation

**Rehypothecation** is the practice of using collateral received from one counterparty to meet collateral demands from another—or more broadly, to fund hedging positions.

Hull defines rehypothecation in *Risk Management and Financial Institutions*: "Rehypothecation occurs when collateral posted by A with B is used by B to meet collateral demands from C." For a dealer, this means collateral received can be used to fund hedge positions. This ability is central to the discounting argument: if you can use received collateral to fund your hedge, your effective funding cost is the collateral remuneration rate.

> **The Lehman Aftermath: Why Rehypothecation Limits Matter**
>
> The Lehman Brothers bankruptcy in September 2008 exposed the risks of unrestricted rehypothecation. Hull notes in *Risk Management and Financial Institutions* Business Snapshot 18.1 that clients who had posted collateral to Lehman's UK prime brokerage "found that they were unable to reclaim their collateral" because it had been rehypothecated.
>
> The consequences were severe: clients became unsecured creditors in bankruptcy proceedings, waiting years to recover a fraction of what they had posted. Post-Lehman, CSA clauses limiting or prohibiting rehypothecation became far more common. The UK and US regulatory frameworks now differ on rehypothecation limits.
>
> **Desk Reality:** If your CSA prohibits rehypothecation, the funding argument for OIS discounting becomes more complex. You receive collateral but cannot use it—so your effective funding cost may be higher than the collateral rate. Some desks apply an FVA adjustment in such cases.

---

## 33.2 The Economic Principle: Discounting Follows the Funding of the Hedge

### 33.2.1 The Core Insight: Collateral Rate = Discount Rate

For a **perfectly collateralized** derivative whose collateral accrues at rate $c(t)$, the clean no-arbitrage present value is obtained by discounting cashflows at $c(t)$.

This is not a modeling choice or regulatory convention—it is a **no-arbitrage requirement**. The argument rests on two facts from the sources:

1. The Fed funds rate is "normally the contractual rate used to accrue interest on posted collateral" (*Interest Rate Modeling*).
2. "The yield curve used for discounting is the risk-free zero curve obtained from the overnight indexed swaps (OIS) market" (*Options, Futures, and Other Derivatives*).

The link between these facts is the funding argument we develop below.

### 33.2.2 The Replication Argument

Consider a derivative with value $V(t)$ to us at time $t$. Under perfect collateralization with cash collateral earning rate $c(t)$, we can construct a self-financing replicating strategy:

**Case 1: The derivative is an asset to us ($V(t) > 0$)**

The counterparty posts collateral $C(t) = V(t)$ to us. We hold this cash and must remunerate it at $c(t)$. Critically, under standard CSA terms, we can **rehypothecate** this cash—use it to fund our hedging positions.

Hull defines rehypothecation in *Risk Management and Financial Institutions*: "Rehypothecation occurs when collateral posted by A with B is used by B to meet collateral demands from C." For a dealer, this means collateral received can be used to fund hedge positions. Our effective funding cost for the hedge is therefore $c(t)$, not our unsecured borrowing rate.

**Case 2: The derivative is a liability to us ($V(t) < 0$)**

We post collateral $|C(t)| = |V(t)|$ to the counterparty. We receive interest at $c(t)$ on the posted amount. The cash we post comes from selling hedge positions (or borrowing at the overnight rate if we can invest at that rate). Our effective funding benefit is again $c(t)$.

**The No-Arbitrage Conclusion:**

If we can fund our hedge at rate $c(t)$ (using rehypothecated collateral or the OIS market), then the appropriate discount rate for pricing must be $c(t)$. Using any other rate creates an arbitrage:

- If we discounted at a rate $r > c(t)$, we would undervalue the derivative relative to what we could replicate it for—and a competitor funding at $c(t)$ would undercut us.
- If we discounted at a rate $r < c(t)$, we would overvalue the derivative—and overpay relative to replication cost.

This is the economic logic behind *Interest Rate Modeling*'s statement that "the Libor rate is no longer a good proxy for a discounting rate on collateralized trades" once Fed funds and Libor diverged materially.

### 33.2.3 Numerical Arbitrage Example

To make the arbitrage argument concrete, consider a simple example:

**Setup:**
- You hold a derivative paying $1{,}000{,}000$ in exactly 1 year
- The trade is perfectly collateralized with cash collateral earning the overnight rate
- OIS discount factor: $P^{\text{OIS}}(0,1) = 0.9750$ (implying ~2.56% OIS rate)
- Suppose you incorrectly discount at LIBOR: $P^{\text{LIBOR}}(0,1) = 0.9700$ (implying ~3.09% rate)

**The Arbitrage:**

Under incorrect LIBOR discounting:
$$PV^{\text{LIBOR}} = \$1{,}000{,}000 \times 0.9700 = \$970{,}000$$

Under correct OIS discounting:
$$PV^{\text{OIS}} = \$1{,}000{,}000 \times 0.9750 = \$975{,}000$$

If a dealer prices at $970{,}000$ (LIBOR discounting) but can fund at OIS, they are **undervaluing** the derivative by $5{,}000$. A competitor who prices correctly can:
1. Sell the derivative at $975{,}000$
2. Invest proceeds at OIS to produce $1{,}000{,}000$ at maturity
3. Pay out the $1{,}000{,}000$ cashflow
4. Keep the funding surplus

The market clears at the discount rate that matches available funding. For collateralized trades, that rate is the collateral rate—typically OIS.

> **Analogy: Replication Funding**
>
> Why does Collateral determine the Discount Rate?
>
> 1.  **The Situation**: You buy a derivative from me. You give me \$1m cash as collateral.
> 2.  **The Obligation**: I have to pay you interest on that \$1m (usually the Fed Funds/OIS rate).
> 3.  **The Opportunity**: I now have \$1m in my hand. I can use this cash to fund the hedge for the derivative I just sold you.
> 4.  **The Cost**: My effective cost of funding this trade is exactly the interest I have to pay you (OIS).
>
> Since my funding cost is OIS, I *must* discount the trade at OIS. If I discounted at Libor (which is higher), I'd be pricing the trade using a cost of funds I don't actually face.

### 33.2.4 The Formal Discounting Identity

*Interest Rate Modeling* emphasizes that in risk-neutral pricing, "the numeraire does not need to be the money market account... a positive process is all that is required." Once a numeraire is specified, values follow from the standard pricing formula.

For a perfectly collateralized derivative, the appropriate numeraire is the **collateral account**—cash accruing at the collateral rate $c(t)$:

$$\beta_c(t) = e^{\int_0^t c(u)\,du}$$

The clean price then takes the form:

$$\boxed{V(0) = \mathbb{E}^{Q^c}\left[\sum_k e^{-\int_0^{t_k} c(u)\,du} \cdot \text{CF}(t_k)\right]}$$

where the expectation is taken under the measure associated with the collateral numeraire.

> **Advanced Note: Numeraire and Measure Theory**
>
> *Interest Rate Modeling* Theorem 1.4.2 provides the foundation: "Suppose that a non-dividend paying asset has a strictly positive price process $X$ in some economy with stochastic discount factor $M$. Then $X$ can be used as a numeraire, with a martingale measure $Q^X$ under which the Radon-Nikodym derivative satisfies $dQ^X/dQ = X(T)/(\beta(T) \cdot X(0))$."
>
> For collateralized derivatives, the collateral account $\beta_c(t)$ serves as the numeraire. The associated measure $Q^c$ is the "collateral measure" under which discounting at $c(t)$ is the correct operation. This is not merely a convenience—it is a change of measure that correctly prices the funding dynamics of the collateralized position.
>
> The practical implication: different CSAs with different collateral currencies correspond to different numeraires and different pricing measures. A USD derivative collateralized in EUR must be priced under the EUR collateral measure, then converted to USD.

**Caveat:** A fully rigorous proof that "perfect collateralization implies collateral numeraire discounting" under all CSA legal and operational details would require specifying close-out conventions, rehypothecation rights, haircuts, and settlement timing. What the sources establish is the market-motivated link: collateral remunerates at overnight rates like Fed funds, and OIS rates provide the proxy "risk-free" curve for discounting in multi-curve practice.

---

## 33.3 From CSA to OIS Discounting

### 33.3.1 What Is an OIS?

An **overnight indexed swap (OIS)** exchanges a fixed rate for the geometric average of overnight rates over the swap's term. Hull defines OIS in *Options, Futures, and Other Derivatives* as a swap where "one party pays interest at a fixed rate... and the other party pays interest calculated from geometric averages of overnight rates during the period."

More precisely, for a single-period OIS from $T_1$ to $T_2$, the floating leg payment is based on the compounded overnight rates:

$$\prod_{i=1}^{N} (1 + r_i \cdot \tau_i) - 1$$

where $r_i$ is the overnight rate for day $i$ and $\tau_i$ is the corresponding day fraction. This geometric compounding captures the actual return from rolling overnight investments.

Hull notes that "OIS rates for maturities one year or less... provide the risk-free zero rates that are equivalent to the underlying overnight rates." The OIS curve is thus a market-implied term structure built from swaps referencing the overnight index—exactly what we need for discounting collateralized derivatives.

### 33.3.2 Why OIS Discounting Became Standard Post-Crisis

The shift from LIBOR discounting to OIS discounting was driven by the breakdown of a previously innocuous assumption.

**Before the crisis:** The spread between Fed funds and 3-month LIBOR was typically single-digit basis points. Andersen and Piterbarg note that "the Fed funds rate used to be closely linked to the overnight Libor rate, with the spread between the two in the single basis points."

**During and after the crisis:** That spread exploded. *Interest Rate Modeling* documents that "the spread between the Fed funds rate and 3 month Libor rate... went up to as much as 275 basis points" after September 2007. This meant that a dealer discounting at LIBOR when funding at overnight rates was systematically mispricing by hundreds of basis points.

**Market response:** The OIS market grew dramatically. *Interest Rate Modeling* cites statistics showing OIS-linked derivatives' share growing from "0.8% to 28%" in one reported period. This growth reflected practitioners recognizing that OIS rates, not LIBOR, should be used for discounting collateralized derivatives.

**Conceptual clarity:** The LIBOR-OIS spread is now understood as largely a **credit spread**—it reflects the risk of lending to banks at term rather than overnight. Hull explains in *Options, Futures, and Other Derivatives* that "for derivatives discounting we need a near-risk-free rate. The OIS zero curve provides this."

### 33.3.3 Building an OIS Discount Curve

Bootstrapping an OIS zero curve follows the same logic as building any zero curve from par instruments:

1. **Short maturities:** Use OIS quotes directly to extract discount factors for tenors up to, say, 2 years
2. **Longer maturities:** Solve for discount factors such that each OIS has zero value at inception

The technical details of curve construction belong in Chapter 18 (OIS Curve Construction). Here we emphasize only that the OIS curve is observable from market quotes and provides the discount factors $P^{\text{OIS}}(0,t)$ used to value collateralized derivatives.

### 33.3.4 SOFR and the Post-LIBOR World

With the transition away from LIBOR, the overnight rate landscape has shifted:

| Currency | Pre-2022 Collateral Rate | Post-LIBOR Collateral Rate |
|----------|--------------------------|----------------------------|
| USD | Fed Funds Effective | SOFR (Secured Overnight Financing Rate) |
| EUR | EONIA | €STR (Euro Short-Term Rate) |
| GBP | SONIA | SONIA (unchanged) |

For collateral discounting purposes:
- Most new USD CSAs reference SOFR as the collateral remuneration rate
- Legacy CSAs that referenced Fed Funds may have been amended or continue on Fed Funds
- The spread between Fed Funds and SOFR is typically small (~5-10bp) but non-zero

The core principle is unchanged: **the discount rate follows the collateral rate**, whatever that rate happens to be called. If your CSA pays SOFR on posted collateral, you discount at SOFR. If it pays Fed Funds, you discount at Fed Funds.

---

## 33.4 Multi-Curve Pricing: Discount vs. Projection

### 33.4.1 The Separation Principle

The crisis forced practitioners to recognize that discounting and rate projection are fundamentally different questions:

- **Discount curve (often OIS):** "At what rate can I fund/invest the replicating hedge?"
- **Projection curve:** "What rate will actually set on the floating leg?"

Hull states this clearly in *Options, Futures, and Other Derivatives*: "The yield curve used for discounting is the risk-free zero curve obtained from the OIS market." But "when the payoffs depend on another yield curve (e.g., the Treasury curve or a curve corresponding to risky borrowing), an analyst has to construct a model incorporating movements in both curves."

Andersen and Piterbarg discuss this extensively under the heading of "separating the discount curves used for Libor projection and for outright discounting." They describe constructing "a pair of curves—a curve for discounting and a curve for projecting 3 month (say) forward Libor rates—in a self-consistent way from the market quotes on deposits, FRAs, swaps with 3 month frequency, and overnight index swaps."

### 33.4.2 Why the Curves Differ

Consider a vanilla interest rate swap where we receive 3-month SOFR term rate and pay fixed. Two curves are needed:

**The projection curve** forecasts what the 3-month SOFR term rate will be at each fixing date. This depends on the market's expectation of future term rates—which includes a term premium and potentially credit/liquidity effects relative to overnight rates.

**The discount curve** determines the present value weights applied to each cashflow. Since the swap is collateralized at the overnight rate, the discount curve is the OIS curve.

Before the crisis, LIBOR was used for both purposes (single-curve world). After the crisis, using LIBOR for discounting when you're actually funding at overnight rates creates systematic mispricing.

### 33.4.3 Numerical Illustration: Single-Curve vs. Multi-Curve

**Setup:**
- Notional: $100{,}000{,}000$
- Annual payments at $t=1,2$
- Pay fixed, receive floating

**Legacy single-curve (LIBOR discounting = LIBOR projection):**

Given discount factors:
- $P^L(0,1) = 0.9700$
- $P^L(0,2) = 0.9400$

Annuity: $A_L = 0.9700 + 0.9400 = 1.9100$

Floating-leg PV (single-curve identity): $PV_{\text{float}}^L = 1 - P^L(0,2) = 0.0600$

Par fixed rate: $K_L = 0.0600 / 1.9100 = 3.141\%$

At $K = 3.141\%$, the swap has zero PV in the single-curve world.

**Multi-curve (OIS discounting, separate projection):**

OIS discount factors:
- $P^{\text{OIS}}(0,1) = 0.9750$
- $P^{\text{OIS}}(0,2) = 0.9500$

OIS annuity: $A_{\text{OIS}} = 0.9750 + 0.9500 = 1.9250$

Projected floating rates (from a separate term rate curve):
- $L_1 = 3.20\%$
- $L_2 = 3.50\%$

Floating PV discounted on OIS:
$$PV_{\text{float}}^{\text{OIS}} = 0.0320 \times 0.9750 + 0.0350 \times 0.9500 = 0.0312 + 0.0333 = 0.0645$$

Fixed PV at $K = 3.141\%$:
$$PV_{\text{fixed}}^{\text{OIS}} = 0.03141 \times 1.9250 = 0.0605$$

Swap PV (receive float, pay fixed):
$$PV^{\text{OIS}} = 0.0645 - 0.0605 = 0.0040$$

In dollars: $0.0040 \times \$100{,}000{,}000 = \$400{,}000$

**Key insight:** The same fixed rate that is "par" under legacy single-curve pricing is **not par** under multi-curve pricing. This difference—hundreds of thousands of dollars on a $100 million swap—is why the industry shifted to multi-curve frameworks.

> **The Swap Paradox**
>
> How can the *same* Swap have two different values?
>
> *   **Pre-2008**: We discounted at LIBOR. We thought the PV was Zero.
> *   **Post-2008**: We discount at OIS. The PV is suddenly \$400,000.
> *   **Who is right?** The OIS valuation is right. The LIBOR valuation was mathematically wrong because it assumed we were funding the trade at LIBOR, when in reality (due to collateral) we were funding it at OIS.
> *   **The Result**: The market realized it had been mispricing trillions of dollars of risk for decades.

---

## 33.5 Multi-Currency CSAs and Cheapest-to-Deliver Collateral

### 33.5.1 The Collateral Currency Choice

Many CSAs allow collateral to be posted in multiple currencies. This creates a **cheapest-to-deliver (CTD) collateral option** that affects pricing.

Consider a USD derivative between two parties where the CSA permits collateral in either USD or EUR. The party posting collateral will choose whichever currency minimizes their funding cost. This choice depends on:

1. The overnight rate in each currency (Fed funds vs. €STR)
2. The FX forward basis between currencies
3. The party's relative funding costs in each currency

### 33.5.2 Why Collateral Currency Matters

When the collateral currency is uncertain, the discount rate becomes the **maximum** of available collateral rates (from the perspective of the collateral poster choosing the cheapest option):

$$c(t) = \max\left(c^{\text{USD}}(t), c^{\text{EUR}}(t) + \text{FX basis adjustment}\right)$$

**Example: USD vs. EUR collateral during negative rate periods**

When EUR rates were deeply negative (2015-2021), posting EUR collateral meant *receiving* a negative rate—effectively paying to post collateral. Counterparties with USD/EUR optionality would post USD, where rates were positive.

**The cross-currency basis complication**

Even comparing overnight rates directly is insufficient. You must account for the cost of converting currencies via FX forwards. The cross-currency basis can make one currency systematically cheaper than another even when spot rates suggest otherwise.

Andersen and Piterbarg discuss cross-currency basis swaps extensively: these instruments "exchange floating rates in different currencies" and reveal the market-implied cost of switching funding currencies. The cross-currency basis spread $b(t)$ captures this cost.

### 33.5.3 Practical Implications

For trades with multi-currency CSAs:
- The "OIS curve" isn't unique—it depends on the collateral currency assumption
- Risk systems must track collateral currency exposure separately
- The effective discount rate may depend on which counterparty is in-the-money (and thus posting collateral)

### 33.5.4 Valuing the CTD Collateral Option

When a CSA permits posting in multiple currencies, the collateral poster holds a **cheapest-to-deliver option**. This option has value—and affects the discount rate used for pricing.

**Framework for CTD Valuation:**

Let $\mathcal{C}$ denote the set of eligible collateral currencies. For each currency $i \in \mathcal{C}$, let:
- $c_i(t)$ = overnight rate in currency $i$
- $b_i(t)$ = cross-currency basis spread to convert to the derivative's native currency

The effective collateral rate is:

$$\boxed{c_{\text{eff}}(t) = \max_{i \in \mathcal{C}}\left(c_i(t) + b_i(t)\right)}$$

The collateral poster will always choose the currency that maximizes their effective rate (minimizes their funding cost when posting).

> **Desk Reality: How Traders Think About CTD Collateral**
>
> "CTD collateral is like a swaption embedded in your CSA. When EUR rates went negative, everyone who could post USD did—instantly. The desks that hadn't modeled the optionality got crushed on their P&L."
>
> **Practical approaches:**
> 1. **Simple:** Assume the current CTD currency persists; use that currency's OIS curve
> 2. **Intermediate:** Scenario analysis across plausible rate environments
> 3. **Full model:** Stochastic model of rates in all eligible currencies with optimal switching
>
> Most dealer desks use approach (2) for risk management and approach (1) for day-to-day pricing, with manual overrides when rates are near switching thresholds.

> **I'm not sure** the provided excerpts contain a complete CTD collateral model. *Interest Rate Modeling* mentions "index-discounting basis" and multi-currency curve construction but does not spell out a full CTD collateral pricing framework with explicit optionality valuation. In practice, some desks model this via an additional basis spread; others use scenario analysis.

---

## 33.6 What Breaks When Collateralization Is Imperfect

### 33.6.1 Sources of Imperfection

The "clean" OIS discounting result assumes perfect collateralization: continuous VM, zero threshold, zero MTA, no settlement lag, no default. Real CSAs violate all of these:

**Threshold and MTA:** Leave residual unsecured exposure up to the threshold amount. If $H = \$5\text{m}$ and current MTM is $\$6.2\text{m}$, the counterparty posts $\$1.2\text{m}$ and $\$5\text{m}$ remains unsecured.

**Margining frequency (not continuous):** If VM is daily or weekly, MTM can move between calls, creating exposure during each margining interval.

**Margin settlement lag / cure period:** Hull's Example 20.1 shows this clearly: with a 20-day cure period, collateral at default equals MTM from 20 days earlier. Exposure equals the MTM change during the cure period (which can be substantial).

**Counterparty default:** If the counterparty actually defaults, remaining exposure after collateral becomes a credit loss.

### 33.6.2 The "Clean + Adjustments" Framework

A practical valuation decomposition consistent with the sources is:

$$\boxed{V_{\text{all-in}} \approx V_{\text{clean (OIS-discounted)}} - \text{CVA} + \text{DVA} + \text{(other adjustments)}}$$

where:
- **CVA (Credit Valuation Adjustment):** PV of expected loss from counterparty default
- **DVA (Debit Valuation Adjustment):** PV of expected loss to counterparty from our default
- **Other adjustments:** Effects of collateral imperfections, funding costs (FVA), capital costs (KVA)

The need to incorporate CVA for uncollateralized or imperfectly collateralized contracts is explicitly noted in *Interest Rate Modeling*: "Uncollateralized derivative contracts are subject to credit risk, and a fully consistent pricing approach needs to incorporate the cost of hedging this risk (the so-called credit valuation adjustment or CVA)."

### 33.6.3 Preview: The CVA Formula

Chapter 34 develops CVA in detail. Here we note only the basic formula from Hull's *Risk Management and Financial Institutions*:

$$\boxed{\text{CVA} = \sum_{i=1}^{n} (1-R) \cdot q_i \cdot v_i}$$

where:
- $q_i$: risk-neutral probability of loss from counterparty default during interval $i$
- $v_i$: PV of expected net exposure (after collateral) at interval midpoint
- $R$: recovery rate

Hull notes that "the $v_i$ are usually calculated using Monte Carlo simulation" and that "dealers may have transactions with thousands of counterparties so that the calculation of the $v_i$ for all of them can be computationally very intensive."

### 33.6.4 Hybrid Discounting for Partial Collateralization

> **Practitioner Note: When Perfect OIS Discounting Doesn't Apply**
>
> For trades with material thresholds, imperfect margining, or mixed collateral/uncollateralized portions, pure OIS discounting is technically incorrect. The exposure up to the threshold is effectively uncollateralized and should be discounted at a rate reflecting unsecured funding.
>
> **A practical hybrid approach:**
>
> $$V = V_{\text{collateralized portion}} + V_{\text{uncollateralized portion}}$$
>
> where:
> - Collateralized portion (MTM above threshold): discount at OIS
> - Uncollateralized portion (up to threshold $H$): discount at unsecured funding rate or apply CVA/FVA
>
> **Example:** $100mm swap with $10mm threshold
> - Expected MTM often exceeds threshold → mostly OIS discounting applies
> - But $10mm is always unsecured → adds CVA and potentially FVA
>
> Most desks simplify by discounting everything at OIS and adding XVA adjustments for the unsecured component. This is an approximation but operationally tractable.

### 33.6.5 Downgrade Triggers and Liquidity Risk

Many CSAs contain **downgrade triggers**—clauses that change collateral requirements if a counterparty's credit rating falls below a specified level. Hull discusses these extensively in *Risk Management and Financial Institutions*.

**How downgrade triggers work:**

A typical clause might specify:
- If rated A or above: threshold = $50 million
- If rated BBB: threshold = $10 million
- If rated below investment grade: threshold = zero (full collateralization required)

When a downgrade occurs, the party must immediately post additional collateral equal to the threshold reduction. This can create severe **liquidity stress** precisely when the firm is already under pressure.

> **The AIG Case: Downgrade Triggers in Action**
>
> Hull describes how AIG's near-collapse in 2008 was accelerated by downgrade triggers. As AIG's credit rating fell, counterparties demanded billions in additional collateral under CSA downgrade provisions. AIG couldn't meet these demands, triggering further rating downgrades in a vicious spiral.
>
> "A company might be required to post more collateral if its credit rating decreases. This is known as a downgrade trigger. Because a company's credit rating tends to worsen at the same time as its liquidity worsens, these types of triggers can lead to serious problems."
>
> The lesson: downgrade triggers create **wrong-way risk** between collateral demands and the firm's ability to fund them. Risk managers must stress-test what happens if ratings fall and collateral demands spike simultaneously.

**Implications for pricing:**

When valuing a portfolio with downgrade triggers:
1. **Scenario analysis:** Model collateral demands under rating downgrade scenarios
2. **Liquidity reserves:** Ensure funding is available for contingent collateral calls
3. **CVA sensitivity:** Recognize that exposure profiles change discontinuously at rating thresholds

---

## 33.7 Price Alignment Interest (PAI)

### 33.7.1 What Is PAI?

**Price Alignment Interest (PAI)** is the daily interest payment on variation margin that ensures the economics of margining align with the derivative's theoretical value. It is the operational mechanism by which the collateral remuneration rate enters the trade's cashflows.

For a cleared trade (or bilateral trade with daily settlement), PAI works as follows:
- Each day, the clearinghouse (or bilateral counterparty) calculates the change in MTM
- VM is exchanged to match this change
- **PAI is paid on the VM balance**, typically at the overnight rate (OIS, SOFR, etc.)

Without PAI, the party receiving VM would have "free money"—cash on which they pay nothing. PAI ensures this doesn't happen.

### 33.7.2 PAI Mechanics

**Daily PAI calculation:**

$$\text{PAI}_t = \text{VM Balance}_{t-1} \times r_{\text{overnight}} \times \frac{1}{360}$$

where the day count is typically ACT/360 for USD.

**Example:**
- VM balance at end of day $t-1$: $+\$10{,}000{,}000$ (we hold collateral)
- Overnight rate: 5.25%
- Day count: ACT/360

$$\text{PAI}_t = \$10{,}000{,}000 \times 0.0525 \times \frac{1}{360} = \$1{,}458.33$$

We **pay** this amount to the counterparty (who posted the VM).

### 33.7.3 PAI vs. Traditional Coupon Accrual

PAI fundamentally changes how swap economics work compared to legacy bilateral trades:

| Aspect | Traditional (No VM/PAI) | Modern (Daily VM + PAI) |
|--------|------------------------|------------------------|
| **Funding of MTM** | Party with positive MTM funds at own rate | VM provides funding at overnight rate |
| **Interest on collateral** | May differ from trade economics | PAI aligns collateral interest with discounting |
| **Effective discounting** | Depends on party's funding | Enforced as overnight rate |

**The key insight:** PAI operationalizes OIS discounting. When you receive PAI at the overnight rate on your VM, the economics are identical to discounting at that rate. This is not coincidence—it's the mechanism that makes the no-arbitrage argument work in practice.

### 33.7.4 PAI and P&L

For traders, PAI shows up as a daily P&L line item:

$$\text{Daily P\&L} = \Delta\text{MTM} + \text{Coupon received} - \text{Coupon paid} + \text{PAI received} - \text{PAI paid}$$

A common mistake is ignoring PAI when computing carry. On a large swap book, daily PAI can be substantial:

**Example:** A desk with $\$500\text{mm}$ net positive VM at 5% overnight rate earns:
$$\text{Annual PAI} \approx \$500{,}000{,}000 \times 0.05 = \$25{,}000{,}000$$

This is real money that affects the economics of holding positions.

> **Desk Reality: PAI Reconciliation**
>
> One of the most common sources of P&L breaks on swap desks is PAI miscalculation or misreconciliation:
>
> - **Rate mismatch:** Your system uses yesterday's SOFR; the clearinghouse uses today's
> - **Day count errors:** Using 365 vs. 360
> - **Balance timing:** EOD vs. SOD VM balance
> - **Settlement lag:** PAI accrues on settled VM, not called VM
>
> If your daily P&L is breaking by small amounts that compound over time, check PAI first.

---

## 33.8 Risk Measurement for OIS-Discounted Portfolios

### 33.8.1 OIS PV01 (Discount Curve Sensitivity)

If discounting is OIS-based, the desk measures sensitivity of PV to shifts in the OIS curve. For deterministic cashflows, a first-order parallel-shift approximation is:

$$\Delta PV \approx -\sum_i t_i \cdot P^{\text{OIS}}(0,t_i) \cdot \text{CF}(t_i) \cdot \Delta y$$

where $\Delta y$ is the rate shift in decimal form (1 bp = 0.0001).

**Sign check:** If rates go up ($\Delta y > 0$), discount factors go down, so PV decreases for positive cashflows—the formula correctly produces a negative $\Delta PV$.

### 33.8.2 Discounting Basis Exposure

A useful risk metric is the **discounting basis** PV difference:

$$\Delta PV_{\text{basis}} = PV_{\text{OIS disc}} - PV_{\text{legacy disc}}$$

Post-crisis, this became a material P&L and risk dimension. Desks that repriced their books from LIBOR discounting to OIS discounting saw large Day-1 P&L impacts—in some cases tens of millions of dollars on large swap books.

### 33.8.3 Collateral Terms as Risk Factors

The CSA parameters are themselves risk factors:
- **Threshold:** Higher threshold increases CVA (more unsecured exposure)
- **Margin frequency:** Less frequent margining increases expected exposure
- **Cure period:** Longer MPOR increases potential exposure at default
- **Collateral rate benchmark:** If the CSA rate changes (e.g., Fed Funds to SOFR), discounting changes

These effects must be captured in stress testing and risk limits.

---

## 33.9 Worked Examples

### Example 33.1: Clean vs. Collateralized PV (Single Cashflow)

**Task:** Price a deterministic cashflow under (i) a "risk-free" curve $r$ and (ii) a collateral rate $c$.

**Given:**
- Cashflow: $\text{CF}(1) = \$1{,}000{,}000$ at $t = 1$
- Discount factors: $P^r(0,1) = 0.9700$, $P^c(0,1) = 0.9750$

**PV under $r$:**
$$PV^r = \$1{,}000{,}000 \times 0.9700 = \$970{,}000$$

**PV under collateral rate $c$:**
$$PV^c = \$1{,}000{,}000 \times 0.9750 = \$975{,}000$$

**Difference:**
$$\Delta PV = \$975{,}000 - \$970{,}000 = \$5{,}000$$

**Sanity check:** Higher discount factor → higher PV for a positive cashflow. ✓

---

### Example 33.2: OIS Discounting Swap PV

**Setup:**
- Notional: $\mathcal{N} = \$100{,}000{,}000$
- Annual payments at $t = 1, 2$
- Pay fixed, receive floating

**Legacy single-curve:**
- $P^L(0,1) = 0.9700$, $P^L(0,2) = 0.9400$
- Annuity: $A_L = 1.9100$
- Floating PV: $1 - P^L(0,2) = 0.0600$
- Par rate: $K_L = 0.0600 / 1.9100 = 3.141\%$

At $K = 3.141\%$, swap PV = 0 in the single-curve world.

**OIS discounting + projection:**
- $P^{\text{OIS}}(0,1) = 0.9750$, $P^{\text{OIS}}(0,2) = 0.9500$
- $A_{\text{OIS}} = 1.9250$
- Projected rates: $L_1 = 3.20\%$, $L_2 = 3.50\%$

$$PV_{\text{float}}^{\text{OIS}} = 0.032 \times 0.975 + 0.035 \times 0.950 = 0.0312 + 0.0333 = 0.0645$$

$$PV_{\text{fixed}}^{\text{OIS}} = 0.03141 \times 1.925 = 0.0605$$

$$PV^{\text{OIS}} = 0.0645 - 0.0605 = 0.0040$$

**In dollars:**
$$\$PV^{\text{OIS}} = 0.0040 \times \$100{,}000{,}000 = \$400{,}000$$

---

### Example 33.3: Discount PV01 (Sensitivity to Discount Curve)

**Setup:**
- Fixed-leg annuity, $\mathcal{N} = \$100{,}000{,}000$, $K = 3.50\%$
- Annual payments at $t = 1, 2, 3, 4, 5$

**Base OIS discount factors:**
- $P_1 = 0.9750$, $P_2 = 0.9500$, $P_3 = 0.9250$, $P_4 = 0.9000$, $P_5 = 0.8750$

**Base annuity:**
$$A = 0.975 + 0.950 + 0.925 + 0.900 + 0.875 = 4.625$$

**Base PV:**
$$PV_{\text{fixed}} = \$100{,}000{,}000 \times 0.035 \times 4.625 = \$16{,}187{,}500$$

**Apply +10 bp parallel shift (linearized approximation):**
$$P_i' \approx P_i(1 - 0.001 \times t_i)$$

- $P_1' = 0.9750 \times 0.999 = 0.9740$
- $P_2' = 0.9500 \times 0.998 = 0.9481$
- $P_3' = 0.9250 \times 0.997 = 0.9222$
- $P_4' = 0.9000 \times 0.996 = 0.8964$
- $P_5' = 0.8750 \times 0.995 = 0.8706$

**New annuity:** $A' = 4.611$

**New PV:** $PV' = \$100{,}000{,}000 \times 0.035 \times 4.611 = \$16{,}139{,}000$

**PV change for +10 bp:**
$$\Delta PV = \$16{,}139{,}000 - \$16{,}187{,}500 = -\$48{,}500$$

**Discount PV01 (per 1 bp):**
$$PV01 \approx -\$4{,}850 \text{ per bp}$$

**Sign check:** Rates up → PV down (negative PV01). ✓

---

### Example 33.4: Collateral Frequency Effect (Daily vs. Weekly)

**MTM values over one week (in $mm):**

| Day | MTM |
|-----|-----|
| 0 | 0.0 |
| 1 | 1.0 |
| 2 | 0.8 |
| 3 | 1.2 |
| 4 | 0.4 |
| 5 | 0.6 |

**Daily VM (collateral = MTM at end of each day):**

Exposure during day $d$ (before end-of-day call): $E_d = \max(V_d - C_{d-1}, 0)$

| Day | Exposure |
|-----|----------|
| 1 | $\max(1.0 - 0.0, 0) = 1.0$ |
| 2 | $\max(0.8 - 1.0, 0) = 0.0$ |
| 3 | $\max(1.2 - 0.8, 0) = 0.4$ |
| 4 | $\max(0.4 - 1.2, 0) = 0.0$ |
| 5 | $\max(0.6 - 0.4, 0) = 0.2$ |

**Max daily-margin exposure:** 1.0 mm

**Weekly VM (collateral stays at $C_0 = 0$ all week):**

$$E_d^{\text{weekly}} = \max(V_d, 0) = V_d^+$$

**Max weekly-margin exposure:** 1.2 mm (Day 3)

**Conclusion:** Less frequent margining increases maximum residual exposure (1.0 mm → 1.2 mm).

---

### Example 33.5: Threshold and MTA Effect

**Given:**
- Current MTM: $V = \$5.0\text{m}$
- Threshold: $H = \$2.0\text{m}$
- MTA: $\$0.5\text{m}$

**Collateral required:**
$$\text{Call} = \max(V - H, 0) = \max(5.0 - 2.0, 0) = \$3.0\text{m}$$

Since $\$3.0\text{m} > \text{MTA}$, the transfer occurs.

**Residual exposure:**
$$E = \max(V - C, 0) = \max(5.0 - 3.0, 0) = \$2.0\text{m}$$

The threshold amount ($2.0\text{m}$) remains unsecured—this is credit exposure.

**Small-move case (MTA blocks transfer):**

If $V = \$2.3\text{m}$:
- Call $= \max(2.3 - 2.0, 0) = \$0.3\text{m} < \text{MTA}$
- No transfer occurs
- Exposure $\approx \$2.3\text{m}$

---

### Example 33.6: Margin Lag (Settlement Lag)

**Setup:**
- Daily VM with one-day settlement lag: collateral today = yesterday's MTM
- MTM path (in $mm): Day 0: 0, Day 1: 5, Day 2: 2, Day 3: 6

**Collateral during day $d$:** $C_d = V_{d-1}$

**Exposure:** $E_d = \max(V_d - V_{d-1}, 0)$

| Day | Exposure |
|-----|----------|
| 1 | $\max(5 - 0, 0) = 5$ |
| 2 | $\max(2 - 5, 0) = 0$ |
| 3 | $\max(6 - 2, 0) = 4$ |

**Max exposure:** 5 mm

This illustrates Hull's cure-period logic: collateral reflects stale MTM, leaving exposure equal to recent MTM increases.

---

### Example 33.7: CVA Calculation (Toy Discrete Sum)

**Given:**
- Recovery: $R = 40\%$ → $(1-R) = 0.6$
- OIS discount factors: $P(0,1) = 0.97$, $P(0,2) = 0.94$, $P(0,3) = 0.90$
- Expected exposures after collateral: $EE(1) = \$2.0\text{m}$, $EE(2) = \$1.5\text{m}$, $EE(3) = \$1.0\text{m}$
- Default probabilities: $\Delta PD_1 = 1.0\%$, $\Delta PD_2 = 1.2\%$, $\Delta PD_3 = 1.5\%$

**Year 1:**
$$0.6 \times 0.97 \times \$2{,}000{,}000 \times 0.01 = \$11{,}640$$

**Year 2:**
$$0.6 \times 0.94 \times \$1{,}500{,}000 \times 0.012 = \$10{,}152$$

**Year 3:**
$$0.6 \times 0.90 \times \$1{,}000{,}000 \times 0.015 = \$8{,}100$$

**Total CVA:**
$$\boxed{\text{CVA} = \$11{,}640 + \$10{,}152 + \$8{,}100 = \$29{,}892}$$

---

### Example 33.8: Collateral Rate Change

**Cashflows:** Receive $\$1{,}000{,}000$ at each of $t = 1, 2, 3$

**Under collateral rate $c_1$ (lower rate, higher DFs):**
- $P^{c_1}(0,1) = 0.975$, $P^{c_1}(0,2) = 0.950$, $P^{c_1}(0,3) = 0.925$

$$PV^{c_1} = \$1{,}000{,}000 \times 2.850 = \$2{,}850{,}000$$

**Under collateral rate $c_2$ (higher rate, lower DFs):**
- $P^{c_2}(0,1) = 0.970$, $P^{c_2}(0,2) = 0.940$, $P^{c_2}(0,3) = 0.910$

$$PV^{c_2} = \$1{,}000{,}000 \times 2.820 = \$2{,}820{,}000$$

**Impact:**
$$\Delta PV = \$2{,}820{,}000 - \$2{,}850{,}000 = -\$30{,}000$$

Higher collateral rate → lower PV for receiving cashflows.

---

### Example 33.9: Multi-Currency CSA (USD vs. EUR Collateral)

**Setup:**
- USD derivative pays $\$10{,}000{,}000$ in 1 year
- CSA allows USD or EUR collateral
- USD OIS: $P^{\text{USD}}(0,1) = 0.9700$ (~3.09% rate)
- EUR OIS: $P^{\text{EUR}}(0,1) = 0.9850$ (~1.52% rate)
- Spot EUR/USD: $S_0 = 1.10$
- 1-year forward: $F = 1.0833$

**Option A: USD Collateral**
$$PV^{\text{USD}} = \$10{,}000{,}000 \times 0.9700 = \$9{,}700{,}000$$

**Option B: EUR Collateral (with CIP holding)**

Effective USD discount factor when posting EUR:
$$P^{\text{EUR→USD}}(0,1) \approx P^{\text{EUR}}(0,1) \times \frac{F}{S_0} = 0.9850 \times \frac{1.0833}{1.10} = 0.9700$$

With CIP holding, the two are approximately equal.

**With cross-currency basis of -25bp (EUR cheaper):**
$$P^{\text{EUR→USD, adj}}(0,1) \approx 0.9700 \times e^{0.0025} = 0.9724$$

$$PV^{\text{EUR, adj}} = \$10{,}000{,}000 \times 0.9724 = \$9{,}724{,}000$$

**Advantage to posting EUR:** $\$24{,}000$

---

### Example 33.10: Self-Financing Replication Walkthrough

**Setup:**
- Bank A sells derivative paying $\$1{,}000{,}000$ to counterparty B in 1 year
- Perfect collateralization at $c = 2.50\%$ (continuous)
- A hedges by buying a zero-coupon bond

**Step 1: A's hedge cost**
$$\text{Bond cost} = \$1{,}000{,}000 \times e^{-0.025} = \$975{,}309.91$$

**Step 2: Collateral flows**

At inception:
- Derivative value to A is $-V(0)$ (liability)
- A posts collateral $|V(0)|$ to B
- B pays premium $P$ to A

For A to be self-financing: $P - C(0) = \text{hedge cost}$

**Step 3: Collateral accrual**

If collateral rate = hedge funding rate = 2.50%, the funding is self-financing.

**Step 4: Terminal flows at $t = 1$**
- A receives $\$1{,}000{,}000$ from bond
- A pays $\$1{,}000{,}000$ to B
- A receives back collateral (including interest)
- Net to A: zero

**No-arbitrage price:**
$$\boxed{V(0) = \$1{,}000{,}000 \times e^{-0.025} = \$975{,}309.91}$$

---

### Example 33.11: EUR vs. USD Collateral with Negative Rates

**Setup:**
- 5-year USD swap, $100mm notional
- CSA allows USD or EUR collateral
- USD OIS curve: flat at 3.00%
- EUR OIS curve: flat at -0.50%
- Cross-currency basis: -30bp (EUR funding cheaper in USD terms)

**Step 1: Calculate effective discount factors**

USD collateral:
$$P^{\text{USD}}(0,5) = e^{-0.03 \times 5} = 0.8607$$

EUR collateral (after basis adjustment):
$$P^{\text{EUR,adj}}(0,5) = e^{-(-0.005 - 0.003) \times 5} = e^{0.04} = 1.0408$$

Wait—this produces a discount factor > 1, which seems wrong. Let's reconsider.

**Correct approach:** The party posting EUR collateral earns -0.50% (i.e., pays 0.50% to post). After converting back to USD via FX forwards with -30bp basis, the effective USD funding rate is approximately:

$$r_{\text{eff}}^{\text{EUR}} = r^{\text{EUR}} + \text{FX forward drift} + \text{basis}$$

Under covered interest parity (with basis):
$$r_{\text{eff}}^{\text{EUR}} \approx r^{\text{USD}} + \text{basis} = 3.00\% - 0.30\% = 2.70\%$$

So posting EUR is 30bp cheaper than posting USD.

**Discount factors:**
- USD collateral: $P^{\text{USD}}(0,5) = 0.8607$
- EUR collateral (USD-equivalent): $P^{\text{EUR,eff}}(0,5) = e^{-0.027 \times 5} = 0.8737$

**PV difference on $1mm cashflow at year 5:**
$$\Delta PV = \$1{,}000{,}000 \times (0.8737 - 0.8607) = \$13{,}000$$

**Conclusion:** The party with CTD optionality benefits by ~$13,000 per $1mm 5-year cashflow by choosing EUR collateral.

---

### Example 33.12: Downgrade Trigger Impact

**Setup:**
- Bilateral swap portfolio MTM: $+\$80\text{mm}$ to us
- Current CSA: Threshold = $50mm (counterparty rated A)
- Current collateral held: $80mm - $50mm = $30mm
- Downgrade trigger: If counterparty falls to BBB, threshold becomes $10mm

**Before downgrade:**
- Unsecured exposure: $50mm (the threshold)
- CVA based on this exposure level

**After downgrade to BBB:**
- New threshold: $10mm
- Required collateral: $80mm - $10mm = $70mm
- Collateral call: $70mm - $30mm = $40mm immediate demand

**Liquidity impact on counterparty:**
The counterparty must post an additional $40mm immediately. If they cannot:
- Potential margin dispute
- Possible default trigger
- Our exposure increases if they fail to post

**CVA impact:**
- Before: $50mm unsecured × (expected default loss)
- After (if posted): $10mm unsecured × (higher expected default loss due to lower rating)

The net CVA may increase even with lower threshold because default probability increased.

---

### Example 33.13: Daily PAI Calculation

**Setup:**
- Cleared USD swap
- VM balance (we hold): $+\$25{,}000{,}000$
- SOFR today: 5.30%
- Day count: ACT/360

**Daily PAI we owe:**
$$\text{PAI} = \$25{,}000{,}000 \times 0.0530 \times \frac{1}{360} = \$3{,}680.56$$

**Monthly PAI (30 days):**
$$\text{PAI}_{\text{month}} \approx \$3{,}680.56 \times 30 = \$110{,}417$$

**Annual PAI:**
$$\text{PAI}_{\text{annual}} \approx \$25{,}000{,}000 \times 0.0530 = \$1{,}325{,}000$$

**P&L implication:** If we're holding $25mm positive MTM on a receiver swap, we earn PAI. But this PAI is offset by the fact that we're implicitly "short" rates on our discount curve exposure. The economics balance out—which is exactly the point of PAI.

---

## 33.10 Practical Notes

### 33.10.1 CSA Terms You Must Know Before Discounting

Before saying "OIS discounting," verify:

1. **Collateral currency** — USD/EUR/GBP/other
2. **Collateral remuneration rate** — which benchmark (Fed funds/SOFR/€STR/SONIA) and any spreads
3. **Margin frequency** — daily/weekly; intraday calls?
4. **Threshold** — amount and whether one-way or two-way
5. **MTA** — minimum transfer amount
6. **Cure period / MPOR** — including dispute time
7. **Eligible collateral** — cash vs. securities, haircuts
8. **Rehypothecation rights** — can received collateral fund hedges?
9. **Close-out convention** — mid-market vs. replacement cost
10. **Downgrade triggers** — how thresholds change with ratings

### 33.10.2 Common Pitfalls

**Mixing up discount and projection curves:** Discounting affects PV weights; projection affects floating cashflows. These are different economic questions.

**Saying "OIS discounting" without specifying details:** Which overnight rate? What if the CSA allows multiple currencies?

**Ignoring thresholds and margin lags:** These create residual unsecured exposure. A "$10m threshold CSA" leaves up to $10m unsecured—that's credit risk.

**Confusing CVA with "just a spread":** CVA is an expectation of discounted loss depending on exposure profiles, collateral terms, and default probabilities—not a simple spread.

**Ignoring PAI in carry calculations:** PAI is real cash flow that affects portfolio economics.

**Underestimating downgrade trigger risk:** Contingent collateral demands can create liquidity crises precisely when least affordable.

### 33.10.3 Verification Tests

**Repricing checks:**
- Rebuild curves → benchmark OIS instruments should reprice to near-zero
- Legacy vs. OIS PV differences should match expected basis PV

**Unit checks:**
- 1 bp = 0.0001 (decimal)
- PV units = currency units

**Sign checks:**
- Discount rates up → PV of positive cashflows down

**Limiting cases:**
- Perfect collateral + no default → clean price = OIS-discounted price
- Zero exposure → CVA = 0

---

## Summary

The shift from LIBOR discounting to OIS discounting reflects a fundamental economic truth: **the discount rate must match the funding rate of the replicating hedge**. For collateralized derivatives, that funding rate is the collateral remuneration rate—typically an overnight rate like Fed funds, SOFR, or €STR.

**Key points:**

1. **Collateral remuneration determines discounting.** If cash collateral earns overnight rates, discount at overnight rates. This is a no-arbitrage requirement, not a convention.

2. **The LIBOR-OIS spread blew out during the crisis.** A spread of 275 bp made ignoring the discounting basis impossible.

3. **Multi-curve pricing is now standard.** OIS for discounting, term rate curves for projection. The curves serve different economic functions.

4. **Imperfect collateral creates exposure.** Thresholds, MTAs, and margin lags leave residual unsecured exposure requiring CVA.

5. **The cure period (MPOR) matters.** Even with zero threshold, settlement lag means collateral at default reflects stale MTM.

6. **CSA terms are product-defining.** You cannot choose a discount curve without knowing collateral currency, remuneration rate, threshold, and rehypothecation rights.

7. **Multi-currency CSAs create CTD optionality.** The collateral poster chooses the cheapest currency, affecting the effective discount rate.

8. **Downgrade triggers add liquidity risk.** Collateral demands can spike precisely when a counterparty is least able to meet them.

9. **PAI operationalizes the funding argument.** Daily interest on VM ensures the economics match the theoretical discounting.

---

## Key Concepts Summary

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| **Credit Support Annex (CSA)** | Annex to ISDA Master Agreement specifying collateral terms | Determines which discount rate applies |
| **Variation Margin (VM)** | Collateral exchanged daily to reflect MTM changes | Reduces credit exposure; ties funding to collateral rate |
| **Collateral Remuneration Rate $c(t)$** | Rate paid on posted cash collateral | Directly determines the appropriate discount rate |
| **OIS (Overnight Indexed Swap)** | Fixed vs. geometric average of overnight rates | Market instrument for extracting risk-free term structure |
| **OIS Discounting** | Using OIS curve to discount collateralized derivatives | Standard post-crisis practice; reflects actual funding |
| **Projection Curve** | Curve for forecasting floating rate payments | Separated from discount curve in multi-curve framework |
| **Threshold** | MTM level below which no VM is required | Creates residual unsecured exposure |
| **Margin Period of Risk (MPOR)** | Time between last collateralization and close-out | Exposure can move even with zero threshold |
| **Rehypothecation** | Using received collateral to meet other collateral demands | Determines whether collateral actually funds hedges |
| **CTD Collateral** | Cheapest-to-deliver collateral in multi-currency CSAs | Creates optionality affecting discount rate |
| **Downgrade Trigger** | CSA clause changing terms when credit rating falls | Creates liquidity risk at worst times |
| **Price Alignment Interest (PAI)** | Daily interest paid on variation margin | Operationalizes OIS discounting in practice |
| **CVA** | PV of expected loss from counterparty default | Adjustment needed when collateral is imperfect |

---

## Notation for This Chapter

| Symbol | Definition |
|--------|------------|
| $V(t)$ | MTM value of derivative to us at time $t$ |
| $C(t)$ | Collateral held by us at time $t$ |
| $c(t)$ | Collateral remuneration rate |
| $P^{\text{OIS}}(0,t)$ | OIS discount factor to time $t$ |
| $H$ | CSA threshold |
| $R$ | Recovery rate |
| $q_i$ | Default probability for interval $i$ |
| $v_i$ | PV of expected exposure at interval $i$ |
| $b(t)$ | Cross-currency basis spread |
| $\beta_c(t)$ | Collateral account numeraire |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is a CSA? | Credit Support Annex to an ISDA Master Agreement; specifies collateral requirements for bilaterally cleared OTC trades |
| 2 | What is variation margin (VM)? | Collateral exchanged daily to reflect MTM changes between counterparties |
| 3 | What is the collateral remuneration rate? | The contractual rate paid on posted cash collateral |
| 4 | Why is Fed funds linked to collateral discounting in USD? | Because Fed funds is "normally the contractual rate used to accrue interest on posted collateral" |
| 5 | What is an OIS? | Overnight indexed swap: exchanges fixed rate for geometric average of overnight rates |
| 6 | What is OIS discounting? | Using OIS rates as proxies for the risk-free rate when valuing derivatives |
| 7 | State the no-arbitrage argument for OIS discounting in one sentence | If your hedge is funded at the collateral rate, you must discount at that rate—otherwise competitors funding at that rate can arbitrage you |
| 8 | What is the discount curve used for? | Determining PV weights (time value of money for the replicating hedge) |
| 9 | What is the projection curve used for? | Forecasting floating cashflows (what rate will actually set) |
| 10 | Why did the industry shift from LIBOR to OIS discounting? | LIBOR-overnight spreads reached 275 bp; LIBOR no longer reflected actual funding costs for collateralized trades |
| 11 | What is a CSA threshold? | MTM level below which VM is not required |
| 12 | What is MTA? | Minimum transfer amount; avoids small collateral transfers |
| 13 | What is the cure period (MPOR)? | Time between last collateralization and close-out; typically 10-20 days |
| 14 | Why does the cure period create exposure even with zero threshold? | Collateral at default reflects stale MTM from 10-20 days earlier |
| 15 | What is rehypothecation? | Using collateral posted by one counterparty to fund positions with another |
| 16 | How did the Lehman bankruptcy affect rehypothecation? | Clients couldn't recover rehypothecated collateral; CSA clauses limiting rehypothecation became common |
| 17 | Give the basic CVA formula | $\text{CVA} = \sum(1-R)q_i v_i$ |
| 18 | What is wrong-way risk? | When exposure increases as counterparty credit deteriorates |
| 19 | In a multi-currency CSA, what determines the effective discount rate? | The cheapest-to-deliver collateral currency (after FX basis adjustment) |
| 20 | What information must you verify before choosing a discount curve? | Collateral currency, remuneration rate, threshold, MTA, eligible collateral, rehypothecation rights |
| 21 | What is Price Alignment Interest (PAI)? | Daily interest payment on variation margin that aligns collateral economics with derivative discounting |
| 22 | How is daily PAI calculated? | $\text{PAI} = \text{VM Balance} \times r_{\text{overnight}} \times \text{day fraction}$ |
| 23 | What is a downgrade trigger in a CSA? | A clause that reduces thresholds (increases collateral requirements) if a party's credit rating falls |
| 24 | Why are downgrade triggers dangerous for liquidity? | They demand more collateral precisely when the firm is under financial stress and least able to fund it |
| 25 | What is CTD collateral optionality? | The right to post collateral in the cheapest available currency under a multi-currency CSA |

---

## Mini Problem Set

**1)** Define CSA, VM, IM, threshold, and MTA.

**Solution:** CSA = Credit Support Annex, governs collateral posting. VM = Variation Margin, collateral exchanged as MTM changes. IM = Initial Margin, posted at inception. Threshold = MTM level below which no VM required. MTA = Minimum Transfer Amount.

---

**2)** Explain why discounting follows the collateral rate for a perfectly collateralized trade.

**Solution:** Under perfect collateralization, received collateral can fund the hedge (rehypothecation). The effective funding cost is therefore the collateral rate $c(t)$. Using any other discount rate creates arbitrage: competitors funding at $c(t)$ could offer better prices.

---

**3)** A cashflow of $2{,}000{,}000$ is paid in 2 years. If $P^{\text{OIS}}(0,2) = 0.94$, compute PV.

**Solution:** $PV = \$2{,}000{,}000 \times 0.94 = \$1{,}880{,}000$

---

**4)** Using $P^L(0,1) = 0.97$, $P^L(0,2) = 0.94$, compute the 2-year annual par swap rate (single-curve).

**Solution:** Annuity $A = 0.97 + 0.94 = 1.91$. Floating PV $= 1 - 0.94 = 0.06$. Par rate $K = 0.06/1.91 = 3.14\%$.

---

**5)** With threshold $H = \$5$m and MTM $V = \$6.2$m, what collateral is required? What is residual exposure?

**Solution:** Collateral $= \max(6.2 - 5.0, 0) = \$1.2$m. Exposure $= V - C = \$5.0$m (the threshold amount).

---

**6)** Explain how a cure period creates exposure even with zero threshold.

**Solution:** Collateral at default reflects MTM from $c$ days earlier (e.g., 20 days). If MTM increased during the cure period, collateral is insufficient. Exposure equals the MTM increase over the cure period.

---

**7)** Compute CVA with $R = 40\%$, $q = 1\%$, $v = \$500{,}000$.

**Solution:** $\text{CVA} = (1-R) \times q \times v = 0.6 \times 0.01 \times \$500{,}000 = \$3{,}000$

---

**8)** Give two reasons why OIS discounting replaced LIBOR discounting post-crisis.

**Solution:** (1) LIBOR-OIS spreads reached 275 bp, making the basis material. (2) LIBOR reflects bank credit risk, not risk-free funding; OIS better represents collateralized funding costs.

---

**9)** What is rehypothecation and why does it matter for the discounting argument?

**Solution:** Rehypothecation is using received collateral to meet other collateral demands or fund hedges. It matters because the discounting argument assumes you can fund hedges at the collateral rate—which requires being able to use received collateral.

---

**10)** In a USD/EUR multi-currency CSA, what determines which collateral currency is "cheaper"?

**Solution:** The combination of (1) overnight rates in each currency, (2) cross-currency basis, and (3) FX forward points. The poster chooses the currency that minimizes their funding cost after accounting for all three factors.

---

**11)** Calculate the daily PAI on a VM balance of $50mm when SOFR = 4.80% (ACT/360).

**Solution:**
$$\text{PAI} = \$50{,}000{,}000 \times 0.0480 \times \frac{1}{360} = \$6{,}666.67$$

---

**12)** A CSA has threshold = $25mm when the counterparty is rated A. If downgraded to BBB, threshold becomes $5mm. Current MTM = $40mm. What additional collateral is required upon downgrade?

**Solution:**
- Before downgrade: Collateral = $40mm - $25mm = $15mm
- After downgrade: Collateral required = $40mm - $5mm = $35mm
- Additional collateral call = $35mm - $15mm = **$20mm**

---

**13)** Why is "OIS everywhere" potentially misleading as a slogan?

**Solution:** "OIS" could mean Fed funds OIS, SOFR OIS, €STR, or SONIA. The discount rate must match the specific collateral remuneration rate in the CSA. A USD trade with EUR collateral earning €STR should not use USD OIS.

---

**14)** How would increasing recovery rate $R$ affect CVA?

**Solution:** CVA $\propto (1-R)$. Higher $R$ means lower loss-given-default, hence lower CVA. If $R$ increases from 40% to 60%, CVA decreases by factor $(1-0.6)/(1-0.4) = 0.67$.

---

## Source Map

### (A) Verified Facts — Directly Supported by Sources

| Fact | Source |
|------|--------|
| CSA is an annex requiring collateral; specifies thresholds, MTAs, haircuts | Hull RM Ch 18-20 |
| Fed funds is contractual rate for collateral remuneration in USD | *Interest Rate Modeling* Section 5.1 |
| Fed funds–LIBOR spread reached 275 bp post-crisis | *Interest Rate Modeling* Section 6.5.3 |
| OIS definition: fixed vs geometric average of overnight rates | Hull OFD Ch 7 |
| OIS rates used as proxies for risk-free rates in derivatives valuation | Hull OFD Ch 7, Glossary |
| Multiple yield curves: OIS for discounting, separate projection | Hull OFD Ch 7; *Interest Rate Modeling* Section 6.5 |
| Cure period (MPOR) typically 10-20 days; creates residual exposure | Hull RM Ch 20, Example 20.1 |
| CVA formula: $\sum(1-R)q_i v_i$ | Hull RM Ch 20 |
| Rehypothecation: using received collateral to meet other demands | Hull RM Ch 18 |
| Post-Lehman: clauses limiting rehypothecation became common | Hull RM Ch 18, Business Snapshot 18.1 |
| EONIA, SONIA as overnight proxies for EUR, GBP | *Interest Rate Modeling* Section 5.1 |
| Downgrade triggers can require additional collateral posting | Hull RM Ch 18-20 |
| AIG's collapse accelerated by downgrade triggers | Hull RM (credit crisis discussion) |
| Change of numeraire theorem | *Interest Rate Modeling* Theorem 1.4.2 |
| Cross-currency basis swaps reveal cost of switching funding currencies | *Interest Rate Modeling* |

### (B) Claude-Extended Content — Practitioner Knowledge

| Content | Context |
|---------|---------|
| PAI mechanics and daily calculation | Extended from general derivatives operations knowledge; operationalizes the funding argument |
| CTD collateral valuation framework | Extended from cross-currency basis principles; practical approaches used on dealer desks |
| Hybrid discounting for partial collateralization | Extended from CVA/FVA principles; practical approximation approach |
| Downgrade trigger liquidity risk discussion | Extended from AIG example; broader implications for risk management |
| P&L reconciliation issues with PAI | Extended from operations knowledge; common desk issues |

### (C) Reasoned Inference — Derived from (A) or (B)

| Inference | Derivation |
|-----------|------------|
| Collateral rate $c(t)$ determines discount rate for perfectly collateralized trades | No-arbitrage: if hedge funded at $c(t)$, replication cost uses $c(t)$; using other rate creates arbitrage |
| Numerical examples (swap PV, discount PV01, PAI) | Direct arithmetic from sourced rate/curve relationships |
| Multi-currency CTD effect | Combining collateral remuneration facts with covered interest parity logic |
| Downgrade trigger impact calculation | Direct application of threshold mechanics under rating change |

### (D) Flagged Uncertainties

| Uncertainty | Reason |
|-------------|--------|
| Complete proof of "perfect collateralization ⇒ collateral numeraire discounting" | Sources establish market practice and motivation; don't provide full theorem covering all CSA legal details |
| Cheapest-to-deliver collateral framework with explicit optionality valuation | *Interest Rate Modeling* mentions index-discounting basis but doesn't spell out complete CTD model with option pricing |
| Exact SOFR CSA amendment mechanics | Sources predate full LIBOR transition |
| FVA framework when rehypothecation is prohibited | Sources note FVA controversy but don't provide complete pricing model for this case |

---

*Chapter 33 establishes why collateral terms determine discount rates. Chapter 34 develops the XVA framework (CVA, DVA, FVA) for trades with imperfect collateralization.*
