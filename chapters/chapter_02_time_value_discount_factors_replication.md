# Chapter 2: Time Value of Money, Discount Factors, and Replication

---

## Fact Classification

### (A) Verified Facts

- A discount factor $d(t)$ is the present value today of one unit of currency received at time $t$
- Discount factors can be extracted from bond prices by writing "price = PV of cash flows" and solving sequentially for $d(\cdot)$ at increasing maturities
- Because of time value of money, discount factors typically decline with maturity (in positive-rate environments)
- **Law of One Price:** absent confounding factors (liquidity, special financing rates, taxes, credit risk), two securities (or portfolios) with identical cash flows should have the same price
- Under continuous compounding, discounting a cash amount $A$ due in $n$ years at rate $R$ uses $Ae^{-Rn}$

### (B) Reasoned Inference

- If the market provides a unique set of discount factors $\{P(0,T_i)\}$ for relevant cash-flow dates $\{T_i\}$, then any deterministic cash-flow stream $\{C_i\}$ has a unique no-arbitrage price:

$$\text{PV}_0 = \sum_i C_i \, P(0,T_i)$$

- This is a linear "pricing kernel" for deterministic cash flows: prices are linear in cash flows and pinned down by the discount factors
- Replication + Law of One Price turns pricing into a mechanical step: replicate the cash flows, then price the replicating portfolio

### (C) Speculation (Clearly Labeled)

- Many modern derivatives desks discount collateralized cash flows using an overnight-risk-free curve (often OIS). This chapter does not implement multi-curve discounting; we only note that "risk-free reference rates created from overnight rates are the ones used in valuing derivatives"

---

## Conventions Used in This Chapter

- We treat the chapter's objects as **default-free and deterministic cash flows** (no optionality yet; no credit risk here)
- We use $P(0,T)$ and $d(T)$ interchangeably for the discount factor at maturity $T$
- For rate summaries, we primarily use **continuous compounding** (because it makes algebra clean and is standard in derivatives texts)
- When using deposit/bill-style money-market quotes in examples, we treat the quoted rate as applying linearly over the accrual year fraction $\tau$ (a stylized, but common, "simple interest over $\tau$" convention)
- **Interpolation** (filling in discount factors between quoted maturities) is deliberately deferred: early chapters solve discount factors at selected maturities, and full curve construction methods come later

---

## Notation Glossary

| Symbol | Definition |
|--------|------------|
| $t$ | Valuation time (here $t=0$) |
| $T$ | Maturity date in years |
| $P(0,T)$ | Time-0 price of a zero-coupon bond paying $1 at $T$ (discount factor) |
| $d(T)$ | Same as $P(0,T)$ |
| $C_i$ | Deterministic cash flow paid at time $T_i$ |
| $\text{PV}_0$ | Present value at time 0 |
| $z(T)$ | Continuously compounded zero (spot) rate for maturity $T$, defined by $P(0,T) = e^{-z(T)T}$ |
| $f(0;T_1,T_2)$ | Continuously compounded forward rate for $[T_1,T_2]$ |
| $F(0;T_1,T_2)$ | Simply compounded forward rate for $[T_1,T_2]$, FRA-style |
| $\tau(T_1,T_2)$ | Year fraction between $T_1$ and $T_2$ |

**Simply Compounded Forward Rate (FRA-style):**

$$F(0;T_1,T_2) = \frac{1}{\tau(T_1,T_2)} \left( \frac{P(0,T_1)}{P(0,T_2)} - 1 \right)$$

---

## Core Concepts

### 1) Time Value of Money

**Formal Definition:**

A unit of currency today is worth more than the same unit received later; the tradeoff across time is summarized by discount factors. Discounting is the operation that maps a future certain cash amount to its present value using a rate/discount factor.

**Intuition:**

Money today can be invested. If you must wait for a dollar, you forgo interest you could have earned.

**Trading / Risk / Portfolio Practice:**

Even for "plain" bonds, every coupon and principal payment is a dated cash flow that must be discounted at its own maturity's discount factor ("each cash flow must be discounted at the factor or rate appropriate for the term of that cash flow").

---

### 2) Present Value (PV)

**Formal Definition:**

For deterministic cash flows $\{(T_i, C_i)\}$, PV is the cash amount today that is equivalent (under market discounting) to receiving those cash flows in the future:

$$\text{PV}_0 = \sum_i C_i \, P(0,T_i)$$

Here $P(0,T_i)$ is "the value today of $1 received at $T_i$".

**Intuition:**

Break a stream into "one-dollar-at-$T_i$" building blocks, value each block, and add.

**Trading / Risk / Portfolio Practice:**

PV is the basic mark-to-market. All later risk measures (DV01, key-rate risk, etc.) are derivatives of PV with respect to curve inputs.

---

### 3) Discount Factors and the Discount Factor Curve

**Formal Definition:**

The discount factor for term $t$, $d(t)$, is the present value of one unit of currency received at $t$.

Equivalently, $P(0,T) = d(T)$ is the price of a zero-coupon bond paying 1 at $T$.

**Intuition:**

- $P(0,T)$ tells you "how many dollars today" you must invest to guarantee $1 at $T$
- Discount factors also encode future values: if $d(T)$ grows to $1 by $T$, then $1 grows to $1/d(T)$

**Trading / Risk / Portfolio Practice:**

- A discount factor curve is the central input to pricing any deterministic cash-flow product (fixed-rate bonds, fixed legs of swaps, FRAs once their payoff is deterministic, etc.)
- Traders talk about "the curve" but PV engines use discount factors under the hood

---

### 4) Zero (Spot) Rates $z(T)$ as Summaries of Discount Factors

**Formal Definition (Continuous Compounding):**

A continuously compounded zero rate $z(T)$ is defined by:

$$P(0,T) = e^{-z(T)T}$$

So:

$$z(T) = -\frac{1}{T} \ln P(0,T)$$

Discounting at a continuously compounded rate $R$ over $n$ years multiplies by $e^{-Rn}$.

**Intuition:**

$z(T)$ is the single constant continuous rate that reproduces the observed discount factor to $T$.

**Trading / Risk / Portfolio Practice:**

Zero rates are common as a quote/plot of the curve, but PV should be done using discount factors (or consistently derived equivalents), because products pay multiple dated cash flows.

---

### 5) Forward Rates $f(0;T_1,T_2)$

**Formal Definition (Continuous Compounding):**

If $R_1$ and $R_2$ are the continuously compounded zero rates to $T_1$ and $T_2$, the continuously compounded forward rate $R_F$ for the period $[T_1,T_2]$ is:

$$R_F = \frac{R_2 T_2 - R_1 T_1}{T_2 - T_1}$$

**Formal Definition (Simple/FRA Compounding):**

The simply compounded forward rate prevailing at $t$ for expiry $T$ and maturity $S$ is:

$$F(t;T,S) = \frac{1}{\tau(T,S)} \left( \frac{P(t,T)}{P(t,S)} - 1 \right)$$

**Intuition:**

- Forward rates are the **implied future borrowing/lending rates** consistent with today's curve
- Under continuous compounding, combining rates over subperiods corresponds to **averaging** over time in the exponent

**Trading / Risk / Portfolio Practice:**

- Forwards are how markets express views: e.g., "buy the 1y1y forward" is a rate-view trade
- Many linear rates products can be valued by forecasting cash flows using forwards and discounting

---

### 6) Law of One Price and No-Arbitrage

**Formal Definition:**

Absent confounding factors (liquidity, special financing rates, taxes, credit risk), two securities/portfolios with exactly the same cash flows must have the same price.

**Intuition:**

Investors care about *what they get*, not *how it's packaged*.

**Trading / Risk / Portfolio Practice:**

- "Rich/cheap" relative value = market price vs model PV under a chosen discount curve
- Deviations can persist due to the very confounding factors listed above (funding, liquidity, taxes, etc.)

---

### 7) Replication and the Replicating Portfolio

**Formal Definition:**

A portfolio **replicates** a target security/claim if it produces the **same cash flows** (in every relevant state of the world, if uncertainty is present). Under no-arbitrage, the target must be priced at the cost of its replicating portfolio. This is the core "arbitrage pricing by replication" logic.

**Intuition:**

If you can build the same future cash flows more cheaply, buy the cheap package and sell the expensive one: you lock in profit today with no net future obligation.

**Trading / Risk / Portfolio Practice:**

- For deterministic cash flows, replication is usually **static**: a coupon bond is replicated by a portfolio of zero-coupon bonds at its payment dates (if such zeros exist / can be synthesized)
- "One way" to value is to compute PV from discount factors, and "another way" is to price the replicating portfolio—both agree when the law of one price holds

---

## Math and Derivations

### 1) PV as a Linear Functional of Deterministic Cash Flows

Consider a finite set of payment dates $0 < T_1 < \cdots < T_n$ and a deterministic cash-flow vector:

$$\mathbf{C} = (C_1, \ldots, C_n)^\top$$

Assume zero-coupon bonds (discount bonds) exist for these maturities, with prices $P(0,T_i)$.

**Replication:**

The portfolio holding $C_i$ units of the $T_i$-maturity zero-coupon bond pays $C_i$ at $T_i$ and nothing elsewhere. Summing across $i$ reproduces the cash-flow stream exactly.

**No-Arbitrage Price:**

By the law of one price, the value today must equal the cost of the replicating portfolio:

$$\boxed{\text{PV}_0(\mathbf{C}) = \sum_{i=1}^{n} C_i \, P(0,T_i)}$$

This is the deterministic-cash-flow "pricing kernel" for this chapter: discount factors are the weights.

**Unit Check:**

- $C_i$: currency
- $P(0,T_i)$: currency-per-currency (dimensionless)
- Product: currency; sum: currency

**Sanity Checks:**

- If $C_k = 1$ and all other $C_i = 0$, then $\text{PV}_0 = P(0,T_k)$ (price of a zero)
- If all $T_i \to 0$, then $P(0,T_i) \to 1$ and PV $\to \sum_i C_i$ (immediate cash is undiscounted)

---

### 2) From Discount Factors to Zero Rates (Continuous Compounding)

Hull's continuous compounding convention gives:

- **Future value:** $A$ invested for $n$ years at continuous rate $R$ grows to $Ae^{Rn}$
- **Discounting:** multiply by $e^{-Rn}$

For a zero-coupon bond paying 1 at $T$, the discount factor is:

$$P(0,T) = e^{-z(T)T}$$

So the implied continuously compounded zero rate is:

$$\boxed{z(T) = -\frac{1}{T} \ln P(0,T)}$$

**Assumptions (Explicit):**

We are using continuous compounding for $z(T)$.

---

### 3) Forward Rates from Zero Rates (Continuous Compounding)

If $R_1$ and $R_2$ are the continuously compounded zero rates for maturities $T_1$ and $T_2$, then the continuously compounded forward rate $R_F$ for $[T_1,T_2]$ satisfies:

$$\boxed{R_F = \frac{R_2 T_2 - R_1 T_1}{T_2 - T_1}}$$

**Derivation Sketch (Intuition-Algebra):**

- \$1 invested to $T_2$ grows to $e^{R_2 T_2}$
- Alternatively, invest to $T_1$ then roll from $T_1$ to $T_2$: growth $e^{R_1 T_1} \cdot e^{R_F(T_2-T_1)}$
- Equate and solve for $R_F$

---

### 4) Forward Rates Directly from Discount Factors (Simple/FRA-Style)

The simply compounded forward rate is defined via discount bonds as:

$$\boxed{F(t;T,S) = \frac{1}{\tau(T,S)} \left( \frac{P(t,T)}{P(t,S)} - 1 \right)}$$

At $t=0$:

$$F(0;T_1,T_2) = \frac{1}{\tau(T_1,T_2)} \left( \frac{P(0,T_1)}{P(0,T_2)} - 1 \right)$$

**Unit Check:**

$\frac{P(0,T_1)}{P(0,T_2)}$ is dimensionless; dividing by $\tau$ gives "per year".

---

### 5) Constructing Discount Factors from Simple Instruments (Bootstrapping Idea)

Extraction of discount factors from coupon bond prices: write PV equations and solve sequentially.

In practice, curve construction uses a family of instruments (deposits, bills, swaps, etc.) and an interpolation rule, but the foundational logic is the same: each instrument gives an equation in the unknown discount factors.

**This Chapter's Stance:**

- We demonstrate "small-instrument-set" discount factor construction (Worked Example A)
- We defer "full discount function" extraction and interpolation choices to later curve chapters

---

## Measurement & Risk (Chapter 2 Scope)

### 1) First-Order Intuition: PV Changes When Rates Move

For deterministic cash flows:

$$\text{PV}_0 = \sum_i C_i P(0,T_i)$$

If we parameterize discount factors via continuously compounded zero rates $z(T)$ as:

$$P(0,T) = e^{-z(T)T}$$

then a small change $\Delta z(T_i)$ at maturity $T_i$ changes that discount factor by:

$$\Delta P(0,T_i) \approx \frac{\partial P}{\partial z} \Delta z = (-T_i) e^{-z(T_i)T_i} \Delta z(T_i) = -T_i P(0,T_i) \Delta z(T_i)$$

So PV changes approximately as:

$$\boxed{\Delta \text{PV}_0 \approx -\sum_i C_i \, T_i \, P(0,T_i) \, \Delta z(T_i)}$$

**Interpretation (No Duration Jargon):**

Longer-dated cash flows (large $T_i$) have bigger PV sensitivity to a given small change in their corresponding rate, because they are discounted for longer.

### 2) Replication Underpins No-Arbitrage Pricing

If a claim's cash flows can be replicated by traded instruments, then by the law of one price their time-0 prices must match (otherwise, an arbitrage exists).

---

## Worked Examples

> **Reminder (Scope):** These are stylized, deterministic examples designed to practice the mechanics. Full curve-building with real market conventions (settlement lags, instrument-specific day counts, holiday calendars, interpolation) is deferred.

### Example A: Build Discount Factors from Simple Deposit-Style Quotes

**Goal:** Given short-dated money-market rates, construct $P(0,T)$ at those maturities and compute implied zero/forward rates.

**Inputs (Assumptions Stated):**

We observe three deposit-style annualized rates (simple over the accrual fraction):

| Maturity | Rate |
|----------|------|
| 3M | $R_{0.25} = 4.80\%$ |
| 6M | $R_{0.5} = 5.00\%$ |
| 12M | $R_{1.0} = 5.30\%$ |

- We take year fractions $\tau(0,T) = T$ for simplicity
- We use the money-market idea that interest for a fraction of a year is computed by multiplying the annual rate by the year fraction

**Step 1: Convert Each Quote to a Discount Factor**

A \$1 deposit that accrues simple interest over $[0,T]$ pays $1 + R_T \cdot T$ at $T$. Therefore:

$$P(0,T) = \frac{1}{1 + R_T T}$$

**Compute:**

$T = 0.25$:
$$P(0,0.25) = \frac{1}{1 + 0.048 \cdot 0.25} = \frac{1}{1.012} = 0.9881423$$

$T = 0.5$:
$$P(0,0.5) = \frac{1}{1 + 0.05 \cdot 0.5} = \frac{1}{1.025} = 0.9756098$$

$T = 1.0$:
$$P(0,1) = \frac{1}{1 + 0.053 \cdot 1} = \frac{1}{1.053} = 0.9496676$$

**Step 2: Implied Continuously Compounded Zero Rates**

Using $P(0,T) = e^{-z(T)T}$ under continuous compounding.

So $z(T) = \frac{1}{T} \ln(1 + R_T T)$.

| Maturity | Calculation | Zero Rate |
|----------|-------------|-----------|
| $T = 0.25$ | $\ln(1.012) \approx 0.01193$ | $z(0.25) \approx 4.7714\%$ |
| $T = 0.5$ | $\ln(1.025) \approx 0.02469$ | $z(0.5) \approx 4.9385\%$ |
| $T = 1.0$ | $\ln(1.053) \approx 0.05164$ | $z(1.0) \approx 5.1643\%$ |

**Step 3: Implied Forward Rate from 6M to 12M**

*Simple (FRA-style) forward using discount factors:*

$$F(0;0.5,1.0) = \frac{1}{0.5} \left( \frac{P(0,0.5)}{P(0,1.0)} - 1 \right) = \frac{1}{0.5} \left( \frac{0.9756098}{0.9496676} - 1 \right) = 0.0546341$$

**Output:** $F(0;0.5,1.0) \approx 5.4634\%$ (simple, annualized)

*Continuous forward via Hull's zero-rate formula:*

$$f(0;0.5,1.0) = \frac{z(1.0) \cdot 1.0 - z(0.5) \cdot 0.5}{1.0 - 0.5} = \frac{0.0516432 - 0.0493852 \cdot 0.5}{0.5} \approx 0.0539012$$

**Output:** $f(0;0.5,1.0) \approx 5.3901\%$ (continuous)

**Sanity Checks:**

- Discount factors are positive ✓
- With positive rates, $P(0,T)$ decreases with $T$ ✓

---

### Example B: Compute PV of a Multi-Cashflow Stream Using Discount Factors

**Goal:** Price a simple 1-year coupon instrument from Example A's discount factors.

**Instrument:** A 1-year note with face 100, paying a 6% annual coupon in two equal semiannual payments:

| Time | Cash Flow |
|------|-----------|
| $T = 0.5$ | Coupon = 3 |
| $T = 1.0$ | Coupon = 3 + Principal = 103 |

**Step-by-Step PV:**

$$\text{PV}_0 = 3 \cdot P(0,0.5) + 103 \cdot P(0,1.0)$$

Insert the discount factors from Example A:

$$\text{PV}_0 = 3 \cdot 0.9756098 + 103 \cdot 0.9496676$$

Compute each term:
- $3 \cdot 0.9756098 = 2.9268293$
- $103 \cdot 0.9496676 = 97.8157644$

Sum:

$$\boxed{\text{PV}_0 = 2.9268293 + 97.8157644 = 100.7426}$$

**Outputs:**

Present value: **100.7426** per 100 face

**Unit/Sanity Checks:**

- If rates were zero, PV would be $3 + 103 = 106$. Here rates are positive, so PV < 106 ✓
- If all discount factors decreased, PV would fall—consistent with interest-rate risk intuition ✓

---

### Example C: Replication / No-Arbitrage Pricing Using Discount Bonds

**Goal:** Price a deterministic claim by explicitly constructing a replicating portfolio of discount bonds.

**Claim (Deterministic Cash Flows):**

| Time | Cash Flow |
|------|-----------|
| $T = 0.5$ | 50 |
| $T = 1.0$ | 105 |

**Replicating Portfolio:**

- Hold **50 units** of the 0.5-year zero (each pays 1 at 0.5)
- Hold **105 units** of the 1-year zero (each pays 1 at 1.0)

This matches the claim's cash flows exactly.

**No-Arbitrage Price (Law of One Price):**

Identical cash flows ⇒ identical price absent confounding factors.

$$V_0 = 50 \, P(0,0.5) + 105 \, P(0,1.0)$$

Insert Example A discount factors:

$$V_0 = 50 \cdot 0.9756098 + 105 \cdot 0.9496676$$

Compute:
- $50 \cdot 0.9756098 = 48.7804878$
- $105 \cdot 0.9496676 = 99.7150997$

Sum:

$$\boxed{V_0 = 48.7804878 + 99.7150997 = 148.4956}$$

**Output:**

Replication price: **148.4956**

**Arbitrage Thought Experiment:**

If the claim traded at 149.20 while replication costs 148.4956, you could sell the claim, buy the replicating portfolio, and lock in 0.7044 today with no future net cash flows—violating the law of one price.

---

## Practical Notes

### Common Quoting Gotchas and Implementation Pitfalls

| Issue | Description |
|-------|-------------|
| **Compounding frequency is a unit** | A quoted "10%" means different things under annual vs semiannual vs continuous compounding; conversion matters |
| **Continuous vs market conventions** | Hull uses continuous compounding widely in derivatives valuation; many cash bond markets quote yields with semiannual compounding |
| **Day-count conventions differ** | In the U.S., money-market instruments commonly use Actual/360; other markets differ |
| **Treasury bill discount yield** | Bills can be quoted on a discount-rate basis (interest as a % of face value), which is not the same as return on price |
| **Curve construction requires interpolation** | Market quotes generally do not provide discount factors at every cash-flow date; practical construction needs an interpolation choice (deferred) |
| **Collateral / OIS discounting** | Post-reform "risk-free reference rates" based on overnight rates are used in valuing derivatives in many contexts; multi-curve frameworks belong to later chapters |

### Verification Tests

**Boundary Checks:**

- $P(0,0) = 1$
- $P(0,T) > 0$ for all $T$

**Monotonicity Checks (Conditional):**

- If rates are nonnegative, expect $P(0,T)$ to be non-increasing in $T$
- If negative rates exist, discount factors can exceed 1 or rise with maturity; monotonicity is no longer a reliable check

**Arbitrage Checks:**

- Reprice each calibration instrument: PV(model) must match market price within tolerance
- If two portfolios have identical cash flows, their modeled PVs must be equal (law of one price test)

**Numerical Stability:**

- Ensure no division-by-zero year fractions
- Use consistent time units (days vs years)

---

## Summary & Recall

### 10-Bullet Executive Summary

1. The discount factor $P(0,T) = d(T)$ is the present value today of \$1 received at time $T$
2. Deterministic cash flows price as $\text{PV}_0 = \sum_i C_i P(0,T_i)$ by replication + law of one price
3. Discount factors can be extracted from traded instruments by writing PV equations and solving sequentially
4. In typical positive-rate settings, discount factors decrease with maturity
5. Under continuous compounding, discounting uses $e^{-RT}$
6. Continuous zero rates summarize discount factors via $P(0,T) = e^{-z(T)T}$
7. Forward rates are implied by today's zero rates; with continuous compounding $R_F = \frac{R_2 T_2 - R_1 T_1}{T_2 - T_1}$
8. FRA-style simple forward rates are $F(t;T,S) = \frac{1}{\tau}\left(\frac{P(t,T)}{P(t,S)} - 1\right)$
9. Yield is a derived summary statistic and can be misleading as a measure of relative value or realized return
10. Full curve-building needs instrument conventions and interpolation choices; we defer those details to later chapters

### Cheat Sheet of Core Formulas

| Formula | Expression |
|---------|------------|
| Discount factor / zero price | $P(0,T) = d(T) = \text{PV}_0(\$1 \text{ at } T)$ |
| PV of deterministic cash flows | $\text{PV}_0 = \sum_i C_i P(0,T_i)$ |
| Continuous discounting | PV of $A$ at $T$ with cont. rate $R$: $Ae^{-RT}$ |
| Zero rate (continuous) | $P(0,T) = e^{-z(T)T}$, $z(T) = -\frac{1}{T}\ln P(0,T)$ |
| Forward rate (continuous, from zero rates) | $f(0;T_1,T_2) = \frac{z(T_2)T_2 - z(T_1)T_1}{T_2 - T_1}$ |
| Forward rate (simple/FRA-style) | $F(0;T_1,T_2) = \frac{1}{\tau(T_1,T_2)}\left(\frac{P(0,T_1)}{P(0,T_2)} - 1\right)$ |
| Law of one price | Same cash flows ⇒ same price (absent confounding factors) |

---

## Flashcards (Q/A)

| # | Question | Answer |
|---|----------|--------|
| 1 | What is $d(T)$? | The present value today of \$1 received at time $T$ |
| 2 | How do you price deterministic cash flows $\{(T_i, C_i)\}$? | $\sum_i C_i P(0,T_i)$ |
| 3 | What is the law of one price? | Identical cash flows imply identical price absent confounding factors |
| 4 | What are examples of confounding factors? | Liquidity, special financing rates, taxes, credit risk |
| 5 | What is $P(0,T)$ economically? | The price of a zero-coupon bond paying \$1 at $T$ |
| 6 | Under continuous compounding, what is the PV of \$A at $T$ discounted at rate $R$? | $Ae^{-RT}$ |
| 7 | Define the continuously compounded zero rate $z(T)$ | The rate such that $P(0,T) = e^{-z(T)T}$ |
| 8 | Define the continuous forward rate for $[T_1,T_2]$ from zero rates | $\frac{R_2 T_2 - R_1 T_1}{T_2 - T_1}$ |
| 9 | Define the simply compounded forward rate from discount factors | $\frac{1}{\tau}\left(\frac{P(0,T_1)}{P(0,T_2)} - 1\right)$ |
| 10 | Why can yield-to-maturity be misleading? | It is only a summary statistic and not a reliable measure of relative value or realized return |
| 11 | How are discount factors extracted from bonds in principle? | Write PV equations for each bond's cash flows and solve sequentially for $d(\cdot)$ |
| 12 | What is replication? | Constructing a portfolio with the same cash flows as the target claim |
| 13 | What happens if a claim's market price differs from replication cost? | An arbitrage trade exists (buy cheap, sell rich), ignoring confounding factors |
| 14 | Why defer interpolation choices? | Full discount-function estimation and interpolation are later curve-building topics; discrete extraction can be limited |
| 15 | What overnight-based discounting preview is mentioned? | Overnight risk-free reference rates are used in valuing derivatives in many contexts |

---

## Mini Problem Set

### Questions

1. **Discount factor basics.** If $P(0,1) = 0.95$, what is the PV of \$200 paid at $T=1$?

2. **Zero rate (continuous).** Using $P(0,2) = 0.90$, approximate the continuously compounded 2-year zero rate $z(2)$ (you may use $\ln(1+x) \approx x - \frac{x^2}{2}$ when needed).

3. **Forward rate (FRA-style).** If $P(0,0.5) = 0.98$ and $P(0,1.0) = 0.95$, compute the simply compounded forward rate $F(0;0.5,1.0)$ with $\tau = 0.5$.

4. **Replication.** A claim pays 10 at $T=1$ and 110 at $T=2$. Express its price in terms of $P(0,1)$ and $P(0,2)$.

5. **Curve shock intuition.** Given cash flows at $T=1,3,5$, which cash flow contributes most to PV sensitivity under a parallel increase in rates? Explain using the first-order approximation.

6. **Law of one price test.** Two portfolios have the same cash flows but trade at different prices. Construct an arbitrage strategy and state the profit at time 0.

7. **Deposit-to-DF conversion.** A 9-month deposit rate is 6% on Actual/360 with $n=270$ days. Using $\tau = 270/360$, compute $P(0,0.75)$ under the simple interest convention.

8. **Forward from zero rates.** Zero rates (continuous) are $z(1) = 4\%$ and $z(2) = 5\%$. Compute the continuous forward rate for year 2, $f(0;1,2)$.

9. **Yield vs discounting.** Explain why a single yield-to-maturity can mislead relative value comparisons across bonds with different coupon structures.

10. **Interpolation preview.** You have discount factors only at 1Y and 2Y but need PV for a cash flow at 1.5Y. List two interpolation approaches you might consider later and one risk each introduces.

### Brief Solution Sketches (1–4 Only)

1. $\text{PV} = 200 \cdot 0.95 = 190$

2. $z(2) = -\frac{1}{2}\ln(0.90) = \frac{1}{2}\ln(1/0.90) = \frac{1}{2}\ln(1.1111) \approx \frac{1}{2}(0.1111 - 0.00617) = 0.0525 \approx 5.25\%$

3. $F = \frac{1}{0.5}\left(\frac{0.98}{0.95} - 1\right) = 2(1.0315789 - 1) = 0.0631579 \approx 6.316\%$

4. $V_0 = 10 \, P(0,1) + 110 \, P(0,2)$. By replication + law of one price, this is the no-arbitrage price.

---

## Source Map

### Directly Source-Backed (Definitions / Key Identities)

- Discount factor definition and basic PV usage
- Extraction of discount factors from bond pricing equations (sequential solving)
- Discount factors typically fall with maturity (positive-rate intuition)
- Law of one price definition and confounding factors
- Continuous compounding and discounting with $e^{-RT}$
- Forward rate from zero rates (continuous): $R_F = \frac{R_2 T_2 - R_1 T_1}{T_2 - T_1}$
- Simply compounded forward rate from discount factors (FRA definition)
- Day-count note for money-market interest accrual (Actual/360 U.S. money market)
- Treasury bill discount quote vs true return illustration
- Yield-to-maturity as a potentially misleading summary
- "Interpolation belongs later" motivation (limitations of discrete extraction)

### Derived (From the Above, Shown Step-by-Step)

- PV linearity and "pricing kernel" interpretation for deterministic cash flows
- First-order PV sensitivity to changes in $z(T)$ via differentiation of $e^{-zT}$
- Numerical examples (A–C) using stylized inputs and consistent conventions

### Uncertain / Convention-Dependent (Flagged)

- The exact mapping from a specific desk's deposit quote to $P(0,T)$ depends on instrument conventions (settlement lag, day count, compounding); we used a simplified "simple interest over $\tau$" convention
- Whether discounting should use OIS vs another curve is product- and CSA-dependent; we only previewed the idea
