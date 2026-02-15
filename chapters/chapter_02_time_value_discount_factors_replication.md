# Chapter 2: Time Value of Money, Discount Factors, and Replication

---

## Introduction

A client asks you to price a 10-year zero-coupon bond. The trade seems simple—no coupons, just a single payment of \$100 at maturity. But before you can quote a price, you must answer a question that sits at the heart of every fixed income calculation: *what is future money worth today?*

This question has consumed economists for centuries, and the answer—the **discount factor**—is the single most fundamental concept in fixed income. Get it right, and you can price any deterministic cash flow stream: bonds, swaps, loans, structured products. Get it wrong, and every valuation you produce becomes suspect.

To see why desk teams care: the PV of a single cash flow \(N\) paid at time \(T\) is \(\text{PV} = N \cdot P(0,T)\). If rates shift and \(P(0,T)\) moves, the dollars move. Under continuous compounding with \(P(0,T)=e^{-y(0,T)\,T}\), a small change \(\Delta y\) in the (continuously compounded) zero rate changes PV by about:
$$\Delta \text{PV} \approx -N \cdot T \cdot P(0,T) \cdot \Delta y$$
So a 1bp move (\(\Delta y=0.0001\)) on a 10-year \(N=\$100\text{mm}\) cash flow can easily be a **tens-of-thousands of dollars** PV move. Multiply that across a trading book and you get the “mystery P&L” that product control spends nights reconciling.

This chapter establishes the foundational machinery for pricing deterministic cash flows:

1. **Discount factors**—the fundamental building blocks that convert future dollars into present value
2. **Present value**—the "pricing kernel" that values any stream of known cash flows
3. **The law of one price and replication**—why identical cash flows must have identical prices, and how arbitrage enforces this
4. **Treasury STRIPS**—zero-coupon securities that make discount factors directly observable
5. **Zero rates and forward rates**—alternative expressions of the time value of money
6. **Modern discounting**—why collateral determines the "risk-free" rate in today's markets
7. **Limits to arbitrage**—when theory meets the reality of balance sheet constraints

Chapter 1 established the conventions—day counts, compounding, settlement. This chapter uses those conventions to build the valuation framework. Together, they provide the foundation for everything that follows: bond pricing in Chapter 5, rate risk (DV01/duration) in Chapters 11–13, curve construction in Part IV, and the multi-curve framework in Chapter 19.

Prerequisites: [Chapter 1 — Market Quoting, Calendars, and Cashflow Plumbing](chapters/chapter_01_market_quoting_calendars_cashflow_plumbing.md)  
Follow-on: [Chapter 3 — Zero Rates, Forward Rates, Par Rates — The Triangle](chapters/chapter_03_zero_forward_par_rates_triangle.md), [Chapter 4 — Money-Market Building Blocks](chapters/chapter_04_money_market_building_blocks.md), [Chapter 5 — Fixed-Rate Bond Pricing](chapters/chapter_05_fixed_rate_bond_pricing.md), [Chapter 11 — DV01/PV01: Definitions and Computation](chapters/chapter_11_dv01_pv01_definitions_computation.md), [Chapter 17 — Curve Construction: Bootstrapping and Interpolation](chapters/chapter_17_curve_construction_bootstrapping_interpolation.md), [Chapter 19 — Projection Curves, LIBOR/SOFR, and the Multi-Curve Framework](chapters/chapter_19_projection_curves_libor_sofr_multi_curve.md)

## Learning Objectives
- Translate a deterministic cashflow into discount factors and present value (\(PV\)).
- Extract (bootstrap) short-maturity discount factors from traded bond prices.
- Explain the law of one price and replication as the core logic behind no-arbitrage pricing.
- Compute a simply compounded forward rate \(F(0;T_1,T_2)\) from discount factors and interpret it as a breakeven future borrowing rate.
- Compute and interpret a first-order rate risk number with an explicit bump object, units, and sign (DV01 preview).

---

## 2.1 Discount Factors: The Price of Future Money

> **The Rosetta Stone of Rates: A Translation Guide**
>
> In fixed income, we often express the same economic reality using three different languages.
>
> 1. **Discount Factor \(P(0,T)\)**: The **price** of a zero-coupon bond.
>    - *Intuition:* "The Price Tag." ($0.95$)
> 2. **Zero Rate \(y(0,T)\)**: The **single rate** over \([0,T]\) that reproduces \(P(0,T)\) under a stated compounding convention.
>    - *Intuition:* "The Sticker Percentage." (e.g., \(5\%\) continuously compounded)
> 3. **Forward Rate \(F(0;T_1,T_2)\)**: The **breakeven rate** you can lock in today for the future period \([T_1,T_2]\).
>    - *Intuition:* "The Future Borrowing Quote."
>
> They are mathematically equivalent—you can convert one to another—but they label the same item differently. We start with the **Discount Factor** because it is the most fundamental: it is a price.

### 2.1.1 Definition

The **discount factor** \(d(t)\) is the present value today of one unit of currency received at time \(t\). We write \(d(T)\) (equivalently \(P(0,T)\)) for the discount factor from today (time 0) to maturity \(T\).

If \(d(0.5) = 0.97557\), then \$1 received in six months is worth 97.557 cents today. Equivalently, \(d(0.5)\) is the price today (per \$1 face) of a zero-coupon bond paying \$1 in six months.

This simple definition has powerful implications. If we know the discount factor for each future date, we can price any security with deterministic cash flows by multiplying each cash flow by its corresponding discount factor and summing them up.

> **Desk Reality: The "Exchange Rate in Time"**
>
> Think of a discount factor as an exchange rate—but instead of converting between currencies, it converts between *times*. Just as the EUR/USD rate tells you how many dollars you get for one euro, the discount factor $P(0,T)$ tells you how many dollars today you get for one dollar at time $T$.
>
> When a PM says "I need the 5-year discount factor," they're asking: "What's the exchange rate between today's dollars and 5-year dollars?" The answer might be 0.78—meaning a dollar in 5 years is worth 78 cents today.

### 2.1.2 Extracting Discount Factors from Bond Prices

Discount factors are not directly quoted in the market—they must be extracted from traded instruments. The example below uses Treasury coupon bonds and works from short maturities to long: solve for the shortest discount factor first, then use it to solve for the next one, and so on.

**Worked example (Treasury bond):** On February 15, 2001, the $7\frac{7}{8}$s of August 15, 2001, traded at $101\text{-}12\frac{3}{4}$ (i.e., $101 + 12.75/32 = 101.3984\%$ of face). This bond pays its final coupon plus principal in six months: $100 + 7.875/2 = 103.9375$. Therefore:

$$101.3984 = 103.9375 \times d(0.5)$$

Solving implies $d(0.5) = 0.97557$.

For a one-year bond—say, the $14\frac{1}{4}$s of February 15, 2002, trading at $108\text{-}31+$ (or $108.984375$)—the cash flows are $7.125$ in six months and $107.125$ in one year. The pricing equation is:

$$108.984375 = 7.125 \times d(0.5) + 107.125 \times d(1)$$

Since we already know $d(0.5) = 0.97557$, we can substitute it in:

$$108.984375 = 7.125 \times 0.97557 + 107.125 \times d(1)$$

$$108.984375 = 6.9509 + 107.125 \times d(1)$$

$$102.0334 = 107.125 \times d(1)$$

$$d(1) = 0.95247$$

**Anchor (price \(\leftrightarrow\) rate):** discount factors are *prices*. If you want to express the same information as a rate, you are just re-parameterizing the same object. For example, under continuous compounding:
$$y(0,T)=-\frac{1}{T}\ln P(0,T)=-\frac{1}{T}\ln d(T).$$

**Check (translate the numbers):** the extracted discount factors translate into annualized rates. Using the formula above:
- \(y(0,0.5)\approx -\ln(0.97557)/0.5 \approx 4.95\%\)
- \(y(0,1.0)\approx -\ln(0.95247)/1.0 \approx 4.87\%\)

This “translation” is useful operationally: if a discount factor looks suspicious, convert it into a rate you can sanity-check. Conversely, if someone gives you a rate quote, convert to \(P(0,T)\) before mixing instruments with different compounding and day-count conventions.

### 2.1.3 Sequential Extraction (Bootstrapping Preview)

**Continuing sequentially**, each additional bond with a longer maturity provides one more equation to solve for one more discount factor. An illustrative set of extracted discount factors out to 2.5 years is:

| Maturity | Discount Factor |
|----------|-----------------|
| 0.5 | 0.97557 |
| 1.0 | 0.95247 |
| 1.5 | 0.93045 |
| 2.0 | 0.90796 |
| 2.5 | 0.88630 |

This sequential approach—solving for one discount factor at a time using known shorter-maturity factors—is the essence of **bootstrapping**, which we formalize in Chapter 17 on curve construction.

### 2.1.4 The Discount Factor Curve

The collection of discount factors across all maturities is called the **discount function** or discount factor curve. With positive rates, discount factors typically fall with maturity: the longer you delay receiving \$1, the less it is worth today.

A plot of \(P(0,T)\) against \(T\) makes this relationship vivid: even “a few percent” rates compound over time, so a long-dated \$1 payment can be worth only a small fraction of a dollar today. This dramatic decline is why discounting matters so much for long-dated cash flows.

### 2.1.5 Discount Factors in Negative Rate Environments

Negative rates do not break the math. **When a (continuously compounded) zero rate is negative, the corresponding discount factor exceeds 1.** For example, if the one-year rate is -0.5%, then:

$$P(0,1) = e^{-(-0.005) \times 1} = e^{0.005} = 1.00501$$

**Economic meaning:** you would have to pay about $1.005 today to receive $1 in one year. In other words, “safe storage of value” can have a price; negative yields are one way markets express that price.

This matters operationally because some “sanity checks” built for positive-rate environments implicitly assume \(P(0,T)\le 1\).

> **Desk Reality: Operational Impacts of Negative Rates**
>
> Negative rates tend to break “folk assumptions” that are hard-coded into spreadsheets and risk systems, for example:
> - assuming \(P(0,T)\le 1\) or assuming discount factors are strictly decreasing;
> - assuming all coupon bonds price below par for low coupons;
> - sanity checks that flag negative forwards as “errors” even when they are economically possible.
>
> The practical lesson is to make your checks conditional on the regime: validate internal consistency first (repricing, curve smoothness, day count correctness), then worry about economic reasonableness.

### 2.1.6 Sanity Checks for Discount Factors

Before using any discount factor curve, verify:

1. **$P(0,0) = 1$**: A dollar today is worth a dollar today
2. **$P(0,T) > 0$**: Discount factors must be positive (no infinite rates)
3. **Monotonicity (positive rates)**: $P(0,T_1) > P(0,T_2)$ when $T_1 < T_2$
4. **Repricing**: Bonds used to extract discount factors should reprice exactly

---

## 2.2 Present Value: The Pricing Kernel for Deterministic Cash Flows

### 2.2.1 The Formula

Given a set of deterministic cash flows $\{C_1, C_2, \ldots, C_n\}$ paid at times $\{T_1, T_2, \ldots, T_n\}$, the present value is:

$$\boxed{\text{PV}_0 = \sum_{i=1}^{n} C_i \, P(0, T_i)}$$

This formula is the "pricing kernel" for all fixed cash-flow instruments: discount each cash flow at its own maturity-specific discount factor, then sum.

The formula works because each cash flow $C_i$ at time $T_i$ can be viewed as $C_i$ units of a zero-coupon bond maturing at $T_i$. The price of that zero is $P(0, T_i)$ per unit, so the price of $C_i$ units is $C_i \times P(0, T_i)$.

### 2.2.2 Linearity

> **Analogy: The Grocery Basket Principle**
>
> Pricing a bond is exactly like pricing a basket of groceries at the checkout counter.
> - **Bond** = 10 Coupons (Apples) + 1 Principal (Steak).
> - **Price** = $10 \times \text{Price}(\text{Apple}) + 1 \times \text{Price}(\text{Steak})$.
>
> You don't need a special "Basket Pricing Theory" to know the total. You just sum the value of the parts. That is exactly what the formula $\sum C_i \, P(0, T_i)$ does. It values each cash flow as a separate item using its specific price tag (the discount factor for that date).

Notice that present value is **linear** in the cash flows: doubling all cash flows doubles the present value. This linearity is fundamental—it means we can decompose complex securities into simpler pieces, value each piece independently, and add the results.

### 2.2.3 Unit Check and Sanity Checks

**Unit check:** $C_i$ is in currency units; $P(0, T_i)$ is dimensionless (currency per currency). The product is currency, and the sum is currency. Correct.

**Sanity checks:**
- If $C_k = 1$ and all other $C_i = 0$, then $\text{PV}_0 = P(0, T_k)$—the price of a zero.
- If all $T_i \to 0$, then $P(0, T_i) \to 1$ and $\text{PV}_0 \to \sum_i C_i$—immediate cash is undiscounted.
- With positive rates, if discount factors fall, PV falls—consistent with rates moving inversely to prices.

---

## 2.3 The Law of One Price and No-Arbitrage

### 2.3.1 The Economic Principle

The **law of one price** says that two portfolios with identical payoffs must have the same price; otherwise, you can sell the expensive version and buy the cheap version for an arbitrage profit. In this chapter we apply that idea to deterministic cash flows: if two instruments have the same contractual cash flows on each date, they should have the same PV (up to frictions).

The intuition is straightforward: an investor should not care whether \$1 on a particular date comes from one bond or another.

This principle has immediate practical implications: once you extract discount factors from one set of traded instruments, you can use those discount factors to price any other instrument with cash flows on the same dates.

### 2.3.2 Confounding Factors

The law of one price holds "absent confounding factors." In practice, several factors can cause prices to deviate:

| Factor | Effect |
|--------|--------|
| **Liquidity / benchmark status** | More liquid “benchmark” bonds can trade rich (higher prices / lower yields) |
| **Special financing (repo specials)** | Bonds in high demand for shorting may trade rich |
| **Taxes** | Different tax treatment affects after-tax returns |
| **Credit risk** | Different default/recovery expectations change required yields (relevant outside government benchmarks) |
| **Balance sheet costs** | Holding positions requires capital and funding |

When prices deviate from the law of one price, practitioners describe securities as trading **rich** (price above fair value) or **cheap** (price below fair value). These deviations create relative value opportunities—though exploiting them requires understanding *why* they exist.

> **Desk Reality: Rich and Cheap in Practice**
>
> When a trader says "the 10-year is trading 3 basis points cheap to the curve," they mean:
> 1. The bond's market yield is 3bp higher than what the fitted curve implies
> 2. The market price is *below* the model price (cheap = low price = high yield)
> 3. There might be a reason (illiquidity, supply pressure, repo dislocation)
>
> Traders don't automatically buy "cheap" bonds. They ask: *why* is it cheap? If it's cheap because of temporary technical factors (auction supply, index rebalancing), they might buy. If it's cheap because it's hard to finance in repo, the "cheapness" might persist indefinitely.

---

## 2.4 Arbitrage and Replication

### 2.4.1 What Is Arbitrage?

An **arbitrage** is a trade with no net investment that never loses money and has a strictly positive gain in at least one scenario. One convenient way to write this idea is:

$$\boxed{\text{Arbitrage: Initial Cost} \leq 0, \; \text{Payoff} \geq 0, \; P[\text{Payoff} > 0] > 0}$$

In words: you pay nothing (or receive cash) upfront, you never lose money, and there's a positive probability of making money. This is a "free lunch"—and in efficient markets, free lunches get eaten quickly.

The existence of arbitrage opportunities is what enforces the law of one price. If two portfolios with identical cash flows traded at different prices, arbitrageurs would buy the cheap one and sell the expensive one, earning a risk-free profit. This trading pressure would quickly eliminate the price difference.

### 2.4.2 Replication: Constructing Identical Cash Flows

**Replication** means constructing a portfolio that produces the same cash flows as a target security. Under no-arbitrage, the target must be priced at the cost of its replicating portfolio.

**Worked example (coupon bond replication):** Consider the $10\frac{3}{4}$s of February 15, 2003. It has four remaining cash flows: $5.375$ at each of 0.5, 1.0, and 1.5 years, and $105.375$ at 2.0 years. Using the discount factors for those maturities (from Section 2.1), you can value the bond directly. You can also replicate its cash flows using a portfolio of shorter-maturity instruments; under no-arbitrage, both approaches must agree.

The key insight of replication is that we can match cash flows at each date by working backward from the final payment.

> **Replication in Action: Matching the Cash Flows**
>
> To replicate a hypothetical bond paying $50 at 0.5y and $1050 at 1.0y, using zeros:
>
> | Maturity | Target Bond CF | Bond A (0.5y Zero) | Bond B (1.0y Zero) | **Portfolio Net CF** |
> | :--- | :--- | :--- | :--- | :--- |
> | **0.5y** | $50 | $50 (50 units) | $0 | **$50** |
> | **1.0y** | $1050 | $0 | $1050 (1050 units) | **$1050** |
>
> *Result:* The Portfolio Net CF column matches the Target Bond CF column exactly. Therefore, the price of the portfolio today **must** equal the price of the target bond.

### 2.4.3 Replication Example: Two Methods Must Agree

**Method 1: Discount Factor Valuation**

Using the discount factors extracted in Section 2.1:

$$V_0 = 5.375 \times 0.97557 + 5.375 \times 0.95247 + 5.375 \times 0.93045 + 105.375 \times 0.90796$$

$$V_0 = 5.244 + 5.120 + 5.001 + 95.676 = 111.041$$

**Method 2: Replicating Portfolio Cost**

The replicating portfolio constructed from the underlying bond set has a total cost of 111.041 (computed by summing the portfolio weights times bond prices).

**The two methods produce identical values.** This is not a coincidence—it is guaranteed by the law of one price. If the discount factors were extracted correctly from the bond prices, both methods are mathematically equivalent.

### 2.4.4 Identifying Rich/Cheap Securities

Suppose the $10\frac{3}{4}$s traded at 110.938 while the replication value is 111.041. The bond is therefore **cheap** by $111.041-110.938 = 0.103$ (about 3.3 32nds).

An arbitrageur could exploit this by:
1. **Buy** the $10\frac{3}{4}$s at 110.938
2. **Sell** the replicating portfolio at 111.041
3. **Pocket** $0.103$ today with zero net future cash flows

In a frictionless world, arbitrage pressure would drive the cheap bond price up and/or the replication cost down until the gap closes.

### 2.4.5 Static vs. Dynamic Replication

For deterministic cash flows, replication is **static**: once you construct the replicating portfolio, you hold it to maturity. The portfolio automatically produces the same cash flows as the target.

For instruments with optionality or rate-dependent cash flows (covered in later chapters on derivatives), replication becomes **dynamic**: the portfolio must be adjusted continuously as rates change.

---

## 2.5 Treasury STRIPS: Directly Observable Discount Factors

### 2.5.1 What Are STRIPS?

Treasury STRIPS (Separate Trading of Registered Interest and Principal of Securities) are zero-coupon securities created by stripping the individual cash flows from Treasury notes and bonds. When a Treasury is stripped, each coupon payment and the principal payment become separate, tradeable securities.

For example, a 10-year Treasury note with semiannual coupons can be stripped into 21 separate securities: 20 coupon strips (one for each semiannual coupon) and 1 principal strip (the final \$100 principal payment per \$100 face).

### 2.5.2 C-STRIPS vs. P-STRIPS

Two common types are:

- **C-STRIPS (Coupon strips)**: Created from the coupon payments. Since many bonds have coupons on the same date (e.g., February 15 and August 15), C-STRIPS are fungible—a coupon strip from one bond is interchangeable with a coupon strip of the same maturity date from another bond.

- **P-STRIPS (Principal strips)**: Created from the principal payment. Unlike coupon STRIPS, principal STRIPS are identified with the specific bond they came from, because reconstituting a particular bond requires the matching principal strip. This is one reason P-STRIPS can “inherit” richness/cheapness from their source bonds.

### 2.5.3 STRIPS and Discount Factors

Because STRIPS are true zero-coupon securities, their prices directly reveal discount factors:

$$P(0, T) = \frac{\text{STRIP Price}}{100}$$

For example, if an August 15, 2031 C-STRIP trades at 63.50, then:

$$P(0, 10.5) = 0.6350$$

This direct observability makes STRIPS valuable for curve construction. However, STRIPS prices may not perfectly match discount factors implied by coupon bonds due to:

- **Liquidity differences**: STRIPS are less liquid than the underlying bonds
- **Reconstitution option**: The ability to recombine STRIPS into whole bonds creates pricing relationships
- **Tax treatment**: Zero-coupon bonds have phantom income (imputed interest) that may be taxed annually

### 2.5.4 STRIPS as Building Blocks

Economically, any coupon Treasury is a **portfolio of STRIPS**: one strip for each coupon date plus a final strip for principal. A 2-year Treasury with semiannual coupons of \(C\) and principal of \(100\) is equivalent to:
- $C$ units of the 0.5-year strip
- $C$ units of the 1.0-year strip
- $C$ units of the 1.5-year strip
- $(C + 100)$ units of the 2.0-year strip

This decomposition is the economic foundation of the present value formula.

### 2.5.5 Stripping and Reconstitution: The Arbitrage Mechanism

The Treasury permits both **stripping** (breaking a bond into its component STRIPS) and **reconstitution** (reassembling STRIPS back into a whole bond). This creates an arbitrage mechanism that keeps STRIPS prices consistent with whole bond prices.

**Stripping arbitrage**: If the sum of STRIPS prices exceeds the whole bond price, professionals can:
1. Buy the whole bond
2. Strip it into components
3. Sell the STRIPS at a profit

**Reconstitution arbitrage**: If the whole bond trades above the sum of its STRIPS, professionals can:
1. Buy the required STRIPS
2. Reconstitute the whole bond
3. Sell the bond at a profit

Depending on relative pricing, professionals can arbitrage by buying bonds and stripping them into STRIPS, or by buying STRIPS and reconstituting whole bonds.

> **Desk Reality: Why Whole Bonds Sometimes Trade Rich**
>
> When investors prefer to hold whole bonds rather than STRIPS (perhaps for liquidity, benchmark eligibility, or operational simplicity), the whole bond can trade rich to its theoretical value. This creates "negative stripping value"—it costs more to buy the bond than to synthesize it from STRIPS.
>
> The persistence of this richness reflects the *value of convenience*. A bond that's benchmark-eligible, liquid in repo, and easy to settle is worth more than a synthetic equivalent that's operationally burdensome.

---

## 2.6 Zero (Spot) Rates: An Alternative Expression

### 2.6.1 Definition

While discount factors directly answer “what is \$1 at time \(T\) worth today?”, investors often prefer to express the time value of money as *rates*. A **zero (spot) rate** \(y(0,T)\) is the single annualized rate over \([0,T]\) that reproduces the discount factor \(P(0,T)\) under a stated compounding convention.

The relationship depends on the compounding convention.

**Continuous compounding** (used extensively in derivatives):

$$\boxed{P(0,T) = e^{-y(0,T)\,T}}$$

Inverting to solve for the rate:

$$\boxed{y(0,T) = -\frac{1}{T} \ln P(0,T)}$$

> **Mental Math Tip: The Rule of 72**
>
> A quick way to estimate doubling time or effective rates:
> $$\text{Rate} \times \text{Years} \approx 72 \implies \text{Doubling}$$
>
> - If the interest rate is 6%, it takes $\approx 12$ years to double your money ($6 \times 12 = 72$).
> - In discount factor terms: If \(y(0,12) = 6\%\) (continuous), then \(P(0, 12) \approx 0.50\).
> - (Exact check: $e^{-0.06 \times 12} = e^{-0.72} \approx 0.487$. Close enough for mental math!)

**Semiannual compounding** (used for U.S. Treasury yields):

$$P(0,T) = \frac{1}{\left(1 + \frac{y_{sa}(0,T)}{2}\right)^{2T}}$$

where \(y_{sa}(0,T)\) denotes the semiannually compounded zero rate.

Under continuous compounding, compounding over \(n\) years multiplies by \(e^{Rn}\) and discounting over \(n\) years multiplies by \(e^{-Rn}\).

### 2.6.2 Why Continuous Compounding?

Continuous compounding is widely used in derivatives pricing texts and code because it makes discounting algebraically simple: products of discount factors become sums in the exponent.

Continuous compounding simplifies the algebra of time and rates. Combining rates over sequential periods amounts to averaging them. If you earn $r_1$ for the first year and $r_2$ for the second year (both continuously compounded), your average rate over two years is simply $(r_1 + r_2)/2$. This additivity makes continuous compounding particularly elegant for forward rate calculations.

### 2.6.3 Conversion Formulas

Conversion between continuous and $m$-times-per-year compounding:

$$\boxed{R_c = m \ln\left(1 + \frac{R_m}{m}\right)}$$

$$\boxed{R_m = m\left(e^{R_c/m} - 1\right)}$$

**Example:** A rate of 10% with semiannual compounding converts to continuous as:

$$R_c = 2 \ln(1 + 0.10/2) = 2 \ln(1.05) = 0.09758 = 9.758\%$$

The continuous rate is lower because more frequent compounding achieves the same terminal value with a slightly smaller quoted rate; the two rates are just different parameterizations of the same discount factor.

---

## 2.7 Forward Rates: Locking in Future Borrowing Costs

### 2.7.1 What Is a Forward Rate?

A **forward rate** is the interest rate agreed today for a loan or investment over a future period \([T_1,T_2]\). Under no-arbitrage, forward rates are not “forecasts”; they are *breakeven rates implied by today’s discount factors*.

### 2.7.2 The Investor's Choice: Spot vs. Forward

The forward-rate idea comes from a simple “two strategies” comparison:

- **Strategy A:** invest from 0 to \(T_2\) in one step.
- **Strategy B:** invest from 0 to \(T_1\), then reinvest from \(T_1\) to \(T_2\).

If markets allow you to lock in the \(T_1 \to T_2\) reinvestment rate today (e.g., with a forward contract or by trading zero-coupon bonds), **no-arbitrage** says Strategies A and B must have the same terminal value. The rate that makes them equal is the forward rate.

### 2.7.3 The Forward Rate Calculation

A standard no-arbitrage identity for the **simple (simply compounded)** forward rate for the future period \([S,T]\) is:

$$\boxed{1+(T-S)\,F(t;S,T)=\frac{P(t,S)}{P(t,T)}}$$

At \(t=0\), and writing the year fraction as \(\tau(S,T)\) (often \(\tau=T-S\) when times are already year-fractions), the same identity is:

$$\boxed{1+\tau(S,T)\,F(0;S,T)=\frac{P(0,S)}{P(0,T)}}$$

Interpretation: the right-hand side is the *growth factor* from \(S\) to \(T\) implied by today’s discount curve. The left-hand side is the growth factor from earning a simple rate \(F\) over the same year fraction.

### 2.7.4 Forward Rates from Spot Rates (Continuous Compounding)

With continuous compounding, it is convenient to define a continuously compounded forward rate \(f_c(0;T_1,T_2)\) for \([T_1,T_2]\). If \(y(0,T_1)\) and \(y(0,T_2)\) are continuously compounded zero rates, then:

$$\boxed{f_c(0;T_1,T_2)=\frac{y(0,T_2)\,T_2-y(0,T_1)\,T_1}{T_2-T_1}}$$

This formula essentially says that the forward rate is the marginal rate required to bridge the gap between the return at \(T_1\) and the return at \(T_2\).

### 2.7.5 Forward Rates from Discount Factors (Simple Compounding)

Solving the identity in Section 2.7.3 for \(F\) gives a practical formula (FRA-style, simple compounding):

$$\boxed{F(t; S, T)=\frac{1}{T-S}\left(\frac{P(t,S)}{P(t,T)}-1\right)}$$

and at \(t=0\):

$$\boxed{F(0; S, T)=\frac{1}{\tau(S,T)}\left(\frac{P(0,S)}{P(0,T)}-1\right)}$$

**Intuition:** The ratio \(P(0,S)/P(0,T)\) is the implied growth factor from \(S\) to \(T\). Subtracting 1 gives total interest over the period, and dividing by the year fraction \(\tau(S,T)\) annualizes it to a simple rate.

### 2.7.6 Locking In the Forward Rate: A Worked Example

The forward rate is not just an abstract calculation—it can be *locked in* using today's bonds. Here's how:

**Goal:** Lock in a *simple forward rate* for the period from \(S=0.5\) to \(T=1.0\).

**Strategy (forward investment replication):**
- At time 0: **sell** 1 unit of the \(S\)-maturity zero and **buy** \(\frac{P(0,S)}{P(0,T)}\) units of the \(T\)-maturity zero (zero net investment).
- At time \(S\): you must **pay** \$1 (to settle the short \(S\)-maturity zero).
- At time \(T\): you **receive** \(\frac{P(0,S)}{P(0,T)}\) dollars.

This is a forward investment of \$1 at \(S\) that grows deterministically to \(\frac{P(0,S)}{P(0,T)}\) at \(T\). By no-arbitrage, the simple forward rate satisfies:

$$1+\tau(S,T)\,F(0;S,T)=\frac{P(0,S)}{P(0,T)}$$

> **Note:** reversing the positions turns this into a forward *borrowing* trade.

> **Desk Reality: FRAs and Forward-Starting Swaps**
>
> This lock-in mechanism is the theoretical foundation of Forward Rate Agreements (FRAs) and forward-starting swaps. When a corporate treasurer wants to hedge future borrowing costs, they're effectively executing this strategy through a derivatives contract.
>
> The forward rate is the breakeven rate implied by today’s discount factors: it is the rate you can lock in today for that future period. Realized future spot rates can differ.

### 2.7.7 Forward Rates and the Spot Rate Curve

There is a useful geometric relationship: when the forward curve is above the spot (zero) curve, the spot curve must be rising; when the forward curve is below the spot curve, the spot curve must be falling (under consistent conventions).

The intuition is that each spot rate is an average of all forward rates up to that maturity. If the next forward rate is above the current average (the spot rate), the new average must rise. If the next forward rate is below, the new average must fall.

---

## 2.8 Modern Discounting: From LIBOR to OIS

### 2.8.1 The Risk-Free Rate Question

The discount-factor framework assumes you know which curve you are discounting on. But what rate is “risk-free”? Historically, many institutions used LIBOR and swap rates as practical proxies for risk-free discounting. After the 2007–2009 crisis, market practice in many derivatives contexts shifted toward OIS-based discounting, especially for collateralized trades.

### 2.8.2 OIS: The Collateralized Benchmark

An **overnight indexed swap (OIS)** is a swap where a fixed interest rate for a period is exchanged for the geometric average of overnight rates during the period. The fixed rate is referred to as the **OIS rate**.

The practical motivation is that overnight-based benchmarks have been treated as closer proxies to “risk‑free” discounting than term unsecured benchmarks like LIBOR.

### 2.8.3 Collateral and the CSA

A **Credit Support Annex (CSA)** specifies the collateral arrangements for an OTC derivative:
- **Variation margin**: Daily cash posted to cover mark-to-market changes
- **Eligible collateral**: What assets can be posted (cash, government bonds, etc.)
- **Thresholds**: Minimum amounts before collateral is required

In many modeling frameworks, a common simplifying assumption is: variation margin is posted in **cash**, and the collateral balance is **remunerated** at an overnight/OIS rate. One argument for using overnight rates as discounting anchors for collateralized trades is that many inter-dealer transactions are collateralized, with the rate paid on collateral being the overnight Fed funds rate (for USD; analogous overnight rates such as EONIA and SONIA are used in other currencies).

Using the wrong discounting assumptions is a common source of valuation differences between systems. In practice, you need to know the collateral terms (CSA/clearing) before you can say what “risk‑free discounting” means for a given trade.

**Mechanics (intuition):** in a fully collateralized trade with frequent variation margin, the position is continuously “reset” by cash collateral. If you are out-of-the-money you post cash and earn the CSA’s collateral remuneration rate on that balance; if you are in-the-money you receive cash and pay interest at that same rate. In that idealized setting, the collateral remuneration rate is the natural “carry rate” for the position, so discounting at some other curve is equivalent to assuming a different interest-on-collateral rule and will show up as a systematic PV difference.

> **Check (toy number):** suppose a collateralized trade pays \$1 in one year and the collateral remuneration rate is 5% (continuous compounding, toy). Discounting at 5% gives \(PV\approx e^{-0.05}=0.9512\). Discounting at 6% gives \(PV\approx e^{-0.06}=0.9418\). The gap is about 0.94 cents per \$1 notional—around \$9.4mm per \$1bn notional—before any convexity or curve-shape effects. This is why “which curve?” is not a detail.

> **Desk Reality: “Which Curve Are You Using?”**
>
> A frequent source of P&L breaks between front office, risk, and product control is that different systems silently use different discounting assumptions. A useful mental model is:
>
> | Trade context | Common discounting anchor (high level) |
> |---|---|
> | Fully collateralized OTC (daily VM) | Overnight/OIS-type curve consistent with collateral rate |
> | Cleared derivatives | Clearinghouse margin/discounting conventions (often OIS-based) |
> | Uncollateralized / weakly collateralized | Credit/funding adjustments become material (CVA/FVA, etc.) |
>
> The point is not that there is one universal curve, but that **discounting is a contract term**. Before arguing about PV differences, confirm the CSA/clearing setup and the curve configuration in each system.

### 2.8.4 Multi-Curve Framework Preview

The shift to OIS discounting created the **multi-curve framework**: we now use *different curves* for discounting and projecting future cash flows.

Conceptually:
- **Discount curve**: used to discount cash flows to PV.
- **Projection curve**: used to generate forward-looking expectations for floating cash flows.

Before the crisis, many practitioners used the same curve for both roles; in modern practice, separating them is common. We return to this in Chapter 19.

---

## 2.9 Limits to Arbitrage: When Theory Meets Reality

### 2.9.1 The Idealized vs. Real Arbitrage

The textbook arbitrage is frictionless: if prices diverge, arbitrageurs instantly exploit the gap, and prices snap back. Reality is messier. Even when a bond looks cheap relative to a replication benchmark, it can remain cheap for a long time—suggesting that some confounding factors inhibit arbitrage activity.

Why does apparent mispricing persist? Because **real arbitrage is not free**.

### 2.9.2 Why “Arbitrage” Can Persist

Even when two portfolios have the *same* contractual cash flows on paper, real-world frictions can keep prices apart. Common examples include:

- **Funding / financing costs** (including repo economics)
- **Liquidity and market impact**
- **Balance-sheet / capital constraints**
- **Counterparty, settlement, and operational constraints**

For this book, the important message is: replication gives a benchmark price in an idealized world; in practice you must ask what frictions are embedded in the traded prices you calibrate to.

> **Desk Reality: “Near-Arbitrage” Still Has Risk**
>
> A trade can be “arbitrage” in cash-flow space and still have meaningful P&L volatility because:
> - convergence timing is uncertain;
> - funding, margin, and repo terms can change;
> - the hedge you use is not perfectly aligned with the instrument you own.
>
> This is why desk risk limits (VaR, stress, drawdown) matter even for relative-value strategies that are “supposed to converge.”

---

## 2.10 PV Sensitivity: A First Look at Rate Risk

Present value is obtained by appropriately discounting all future cash flows: multiply each cash flow by its discount factor and sum the discounted values.

We now consider the sensitivity of price (present value) to a **parallel shift** of the yield curve. Suppose deterministic cashflows \(x_i\) are paid at times \(t_i\) and discounted at continuously compounded spot rates \(r_i\). Then:

$$PV(0)=\sum_{i=0}^{n} x_i e^{-r_i t_i}$$

Under a parallel shift by \(\lambda\), each spot rate becomes \(r_i+\lambda\), so:

$$PV(\lambda)=\sum_{i=0}^{n} x_i e^{-(r_i+\lambda) t_i}$$

We then differentiate to find (at \(\lambda=0\)):

$$\left.\frac{dPV(\lambda)}{d\lambda}\right|_{\lambda=0}=-\sum_{i=0}^{n} t_i x_i e^{-r_i t_i}$$

In this chapter’s discount-factor notation, the same idea can be written as:

$$PV_0=\sum_i C_i\,P(0,T_i),\qquad y_\lambda(0,T):=y(0,T)+\lambda$$

$$PV(\lambda)=\sum_i C_i \, e^{-(y(0,T_i)+\lambda)\,T_i}$$

and for a small parallel shift \(\Delta y\) in continuously compounded zero rates:

$$\boxed{\Delta PV \approx -\sum_i C_i \, T_i \, P(0, T_i) \, \Delta y}$$

### Link to Duration (Fisher–Weil Intuition)

One useful duration concept here is the **Fisher–Weil duration**, the present-value-weighted average cashflow time:

$$\boxed{D_{FW}:=\frac{1}{PV(0)}\sum_{i=0}^{n} t_i\,x_i\,e^{-r_i t_i}}$$

It has units of time and, when all cashflows are nonnegative, lies between the earliest and latest payment times.

In this chapter’s notation, the same quantity is:

$$\boxed{D_{FW}:=\frac{1}{PV_0}\sum_i T_i\,C_i\,P(0,T_i)}$$

Then the **relative** PV sensitivity to a parallel shift satisfies:

$$\boxed{\frac{1}{PV(0)}\left.\frac{dPV(\lambda)}{d\lambda}\right|_{\lambda=0}=-D_{FW}}$$

### DV01 (Preview): Bump Object, Units, and Sign

One common analytical definition of yield-based DV01 (when the bumped “rate factor” is the bond’s yield \(y\)) is:

$$DV01 = -\frac{1}{10{,}000}\,\frac{dP}{dy}$$

where \(P\) is the bond price (PV).

Throughout this book we use an equivalent finite-difference convention, for a **stated bump object**:

$$\boxed{DV01 := PV(\text{rates down }1\text{bp})-PV(\text{base})}$$

To make that definition operational, you must specify the **bump object**. For this chapter’s deterministic-cashflow setting, use:
- **Bump object:** parallel shift of the continuously compounded zero curve \(y(0,T)\) by \(-1\) bp at all relevant maturities.
- **Units:** currency per 1bp, for the stated notional.
- **Sign:** for a long position with positive cashflows, \(DV01>0\) (rates down \(\Rightarrow\) PV up).

Under the first-order approximation above, this bump gives:

$$\boxed{DV01 \approx \left(\sum_i C_i\,T_i\,P(0,T_i)\right)\times 10^{-4}}$$

**Interpretation:** longer-dated cash flows (larger \(T_i\)) typically contribute more to rate risk. This relationship is the foundation of **duration** (Chapter 11).

> **Pitfall — “DV01” without a bump object:** Two systems can both report “DV01” and still materially disagree because they bump different things (yield vs. zero rates vs. par quotes, different node designs, different day counts).
> **Why it matters:** hedge ratios and P&L explain break (wrong sign or wrong magnitude).
> **Quick check:** for a long bond, rates down should increase PV. If your “DV01” comes out negative, you likely flipped the sign or bumped the wrong object.

---

## 2.11 Worked Examples

### Example 0 (Desk Template): PV and DV01 of a Zero-Coupon Cashflow

**Context**
- A trader buys a Treasury principal STRIP (a zero-coupon bond). You need PV and a first risk number for the risk report.

**Timeline (Make Dates Concrete)**
- Trade date: 2026-02-17
- Settlement date: 2026-02-17 (simplifying assumption; see Chapter 1/5 for settlement conventions)
- Accrual start/end: n/a (zero-coupon)
- Payment date(s): 2031-02-17

**Inputs**
- Instrument: \$100 face paid at maturity (zero-coupon).
- Notional (face): \(N=\$100{,}000{,}000\).
- Quote (hypothetical): continuously compounded 5-year zero rate \(y(0,5.0)=4.20\%\).
- Compounding: continuous; treat \(T=5.0\) years for this toy example.

**Outputs (What You Produce)**
- Discount factor: \(P(0,5.0)\).
- PV: \(PV_0\) in dollars.
- Risk: \(DV01\) with bump object “parallel \(-1\) bp shift in \(y(0,T)\)”.

**Step-by-step**
1. **Quote \(\to\) discount factor:** \(P(0,5.0)=e^{-0.042\times 5}=e^{-0.21}\approx 0.8106\).
2. **PV:** \(PV_0=N\cdot P(0,5.0)\approx \$100{,}000{,}000\times 0.8106=\$81.06\text{mm}\).
3. **DV01 (book convention):** \(DV01 := PV(\text{rates down }1\text{bp})-PV(\text{base})\).
   - Exact: \(PV_{\downarrow 1bp}=N\cdot e^{-(0.042-0.0001)\times 5}\), so \(DV01=PV_{\downarrow 1bp}-PV_0\approx \$40.5\text{k}\).
   - First-order check: \(DV01\approx PV_0\times T\times 10^{-4}\approx 81.06\text{mm}\times 5\times 10^{-4}=\$40.5\text{k}\).

**Cashflows (table)**
| Date | Cashflow | Explanation |
|---|---:|---|
| 2031-02-17 | \(+\$100{,}000{,}000\) | principal STRIP maturity payment |

**P&L / Risk Interpretation**
- A risk report \(DV01\approx \$40.5\text{k/bp}\) means a 10bp rally (rates down) gives about \(+\$405\text{k}\) of PV change (first order), before convexity and curve-shape effects.
- This is why “just one future cashflow” can dominate book risk when it is large and long-dated.

**Sanity Checks**
- Units: \(P(0,T)\) is unitless; \(PV\) is currency; \(DV01\) is currency per 1bp. ✓
- Sign: long STRIP \(\Rightarrow\) rates down increases PV \(\Rightarrow DV01>0\). ✓
- Limit: as \(T\to 0\), \(DV01\to 0\). ✓

---

### Example A: Building Discount Factors from Money-Market Quotes

**Goal:** Given deposit-style annualized rates, construct discount factors and compute implied zero and forward rates.

**Inputs:** We observe three money-market rates (simple interest over the accrual fraction):

| Maturity | Rate (Simple) |
|----------|---------------|
| 3M (0.25Y)| 4.80% |
| 6M (0.50Y)| 5.00% |
| 12M (1.00Y)| 5.30% |

Using year fractions $\tau = T$ and the simple interest convention, a deposit of $1$ grows to $1 + R_T \cdot T$ at maturity.

**Step 1: Discount Factors**
The discount factor is the reciprocal of the growth factor: $P(0,T) = \frac{1}{1 + R_T \cdot T}$.

| Maturity | Calculation | Discount Factor |
|----------|-------------|-----------------|
| 0.25 | $1/(1 + 0.048 \times 0.25)$ | 0.98814 |
| 0.50 | $1/(1 + 0.050 \times 0.50)$ | 0.97561 |
| 1.00 | $1/(1 + 0.053 \times 1.00)$ | 0.94967 |

**Step 2: Continuously Compounded Zero Rates**
Using \(y(0,T) = -\frac{1}{T} \ln P(0,T)\):

| Maturity | Zero Rate (continuous) |
|----------|------------------------|
| 0.25 | 4.77% |
| 0.50 | 4.94% |
| 1.00 | 5.16% |

**Step 3: Forward Rate from 6M to 12M**
Simple (FRA-style):
$$F(0; 0.5, 1.0) = \frac{1}{0.5}\left(\frac{0.97561}{0.94967} - 1\right) = 2 \times (1.02731 - 1) = 5.46\%$$

Continuous:
$$f_c(0; 0.5, 1.0) = \frac{0.0516 \times 1.0 - 0.0494 \times 0.5}{1.0 - 0.5} = \frac{0.0516 - 0.0247}{0.5} = 5.38\%$$

**Sanity checks:**
- Discount factors are positive and decrease with maturity ($0.988 > 0.975 > 0.949$). ✓
- Zero rates increase with maturity (upward-sloping curve). ✓
- The forward rate (5.46%) exceeds the six-month spot rate (5.00%)—consistent with the rising spot curve. ✓

---

### Example B: Pricing a Coupon Instrument

**Goal:** Price a 1-year note paying a 6% annual coupon semiannually.

**Cash flows:**
- Time 0.5: coupon of 3 (since \(6\%/2 \times 100 = 3\) per 100 face)
- Time 1.0: coupon + principal of 103 (per 100 face)

**Calculation:**
Using the discount factors from Example A:

$$\text{PV}_0 = 3 \times 0.97561 + 103 \times 0.94967$$
$$\text{PV}_0 = 2.927 + 97.816 = \boxed{100.74}$$

**Interpretation:** The bond is worth 100.74 per 100 face. Since this exceeds par, the coupon rate (6%) must be higher than the prevailing market yield (which we saw was around 5-5.3%).

**Sanity check:** If rates were zero, PV would be $3 + 103 = 106$. With positive rates, PV must be less than 106. Determining it is $100.74$ passes this check. ✓

---

### Example C: Replication and No-Arbitrage Pricing

**Goal:** Price a claim paying 50 at $T = 0.5$ and 105 at $T = 1.0$ by constructing a replicating portfolio.

**Replicating portfolio:**
- 50 units of the 6-month zero (each pays 1 at \(T=0.5\))
- 105 units of the 1-year zero (each pays 1 at \(T=1.0\))

This portfolio produces exactly the claim's cash flows.

**No-arbitrage price:**
Cost of replication = $50 \times P(0, 0.5) + 105 \times P(0, 1.0)$
$$V_0 = 50 \times 0.97561 + 105 \times 0.94967 = 48.78 + 99.72 = \boxed{148.50}$$

**Arbitrage thought experiment:** If the claim traded at 149.50 while replication costs 148.50, you could:
1. Sell the claim at 149.50 (inflow)
2. Buy the replicating portfolio at 148.50 (outflow)
3. Pocket $1.00$ today
4. At $T=0.5$ and $T=1.0$, the portfolio payoffs exactly cover the claim liabilities.

You have locked in a risk-free profit of $1.00$ with zero net future obligations. Market forces (arbitrageurs executing this trade) would drive the claim price down and/or the component prices up until the gap closes.

---

### Example D: STRIPS Pricing Check

**Goal:** Verify consistency between STRIPS prices and discount factors.

**Given:** From the bootstrapped discount factors above, $d(1.0) = 0.95247$.

**Expected STRIPS price:** A 1-year C-STRIP should trade at $95.247$ per $100 face.

**Verification:** If the STRIPS traded at $95.50$, it would be **rich** by $0.253$. Arbitrageurs could:
1. Strip a coupon bond to create the synthetic zero
2. Sell the C-STRIP at $95.50$
3. Pocket the difference

In practice, STRIPS often trade slightly rich or cheap due to liquidity and tax effects, but large deviations are arbitraged away.

---

### Example E: Locking in a Forward Rate

**Goal:** Show how to lock in the 6M×12M forward rate using today's bonds.

**Given:** $P(0, 0.5) = 0.97561$, $P(0, 1.0) = 0.94967$

**Forward rate:** $F(0; 0.5, 1.0) = 5.46\%$

**Lock-in strategy:**
1. Sell $0.97561/0.94967 = 1.0273$ units of the 1-year zero
2. Buy 1 unit of the 6-month zero

**Cash flow verification:**

| Time | 6M Zero | 1Y Zero | Net |
|------|---------|---------|-----|
| Today | $-0.97561$ | $+1.0273 \times 0.94967 = +0.97561$ | **$0$** |
| 6M | $+1$ | $0$ | **$+1$** |
| 12M | $0$ | $-1.0273$ | **$-1.0273$** |

**Interpretation:** You pay $0$ today, receive $1$ at 6M, and repay $1.0273$ at 12M. The implied interest rate for the 6-month period is:

$$\frac{1.0273 - 1}{0.5} = 5.46\%$$

This exactly equals the forward rate—confirming the lock-in works.

---

## 2.12 Practical Notes

### Common Pitfalls

| Issue | Description |
|-------|-------------|
| **Compounding mismatch** | A "5%" rate means different things under different compounding (e.g., continuous vs. semiannual). Always verify the convention. |
| **Day count ignored** | Discount factor extraction depends on year fractions $\tau$, which depend on day counts (e.g., ACT/360 vs. 30/360). |
| **Wrong discount curve** | Discounting must be consistent with the contract’s collateral/clearing terms (often OIS-type for cash-collateralized trades). |
| **Interpolation deferred** | Market quotes don't provide discount factors at every single date; interpolation methods (Chapter 17) fill the gaps. |
| **Confounding factors ignored** | Rich/cheap analysis requires understanding *why* prices deviate from the model. Is it liquidity? Credit? |
| **STRIPS vs. synthetic zeros** | STRIPS prices may differ from discount factors implied by coupon bonds due to liquidity and tax differences. |
| **Ignoring balance sheet costs** | A "profitable" arbitrage may be unprofitable after funding, capital, and operational costs. |

### Verification Tests

**Boundary checks:**
- $P(0, 0) = 1$ (A dollar today is worth a dollar today)
- $P(0, T) > 0$ for all sensible finite rates.

**Monotonicity (positive rates):**
- $P(0, T)$ should decrease as $T$ increases.

**Arbitrage test:**
- Reprice calibration instruments using your derived curve: the model PV must match the market price exactly.
- Two portfolios with identical cash flows must have identical model PVs.

---

## Summary

This chapter established the fundamental pricing framework for deterministic cash flows:

1. **Discount factors** \(P(0,T)\) give the present value today of \$1 paid at time \(T\). They can be extracted from traded instruments by solving pricing equations.

2. **Present value** is computed as $\text{PV}_0 = \sum_i C_i P(0, T_i)$—multiply each cash flow by its discount factor and sum. This is the "pricing kernel" for fixed cash flows.

3. **The law of one price** states that identical cash flows must have identical prices, absent confounding factors (liquidity, repo specials, taxes, credit, balance sheet).

4. **Replication** constructs a portfolio with the same cash flows as a target. Under no-arbitrage, the target's price must equal the replication cost.

5. **Treasury STRIPS** are zero-coupon securities that make discount factors directly observable, though liquidity and tax effects can cause small deviations.

6. **Zero rates** are an alternative expression: \(P(0,T)=e^{-y(0,T)\,T}\) under continuous compounding. They re-parameterize the same discount factor curve.

7. **Forward rates** are implied future borrowing costs that can be locked in using today's bonds. They link spot rates at adjacent maturities.

8. **Modern discounting** depends on collateral terms: a CSA/clearing setup often motivates discounting using an OIS-type curve consistent with collateral remuneration.

9. **Limits to arbitrage** mean that theoretical mispricings can persist due to balance sheet constraints, funding costs, and liquidity gaps. Real arbitrage is never truly "free."

10. **Rate sensitivity**: PV changes roughly linearly for small rate moves, and the book’s DV01 convention makes the bump object, units, and sign explicit—this is the foundation of duration.

The key conceptual insight: **arbitrage and replication turn pricing into a mechanical step**. If you can match a security's cash flows using traded instruments, the law of one price pins down its value. But the real world adds friction—and understanding those frictions separates textbook pricing from desk-level reality.

---

## Key Concepts

| Concept | Definition | Why It Matters |
|---------|------------|----------------|
| Discount factor \(P(0,T)\) | PV today of \$1 at time \(T\) | The fundamental building block for pricing. |
| Present value | $\sum_i C_i P(0,T_i)$ | The price of any deterministic cash-flow stream. |
| Law of one price | Same cash flows → same price | Justifies using one curve to price all instruments. |
| Replication | Portfolio matching target's cash flows | Provides the proof for the no-arbitrage price. |
| STRIPS | Zero-coupon securities from stripped Treasuries | Make discount factors directly observable. |
| Zero rate \(y(0,T)\) | Rate such that \(P(0,T)=e^{-y(0,T)\,T}\) (continuous) | A convenient way to quote the same discount curve. |
| Forward rate \(F(0;T_1,T_2)\) | Breakeven simple rate for \([T_1,T_2]\) implied by discount factors | Turn discount factors into future-period borrowing/lending rates. |
| OIS discounting | Using overnight rates for collateralized PV | The post-2007 shift for many derivatives contexts. |
| CSA | Credit Support Annex specifying collateral terms | Determines which discount curve to use. |
| Limits to arbitrage | Constraints preventing perfect price enforcement | Balance sheet, funding, liquidity, counterparty. |
| DV01 (book convention) | \(PV(\text{rates down }1\text{bp})-PV(\text{base})\) for a stated bump object | Forces unit/sign clarity and supports hedge sizing. |

---

## Notation

| Symbol | Meaning | Units / Convention |
|---|---|---|
| \(P(0,T)\), \(d(T)\) | discount factor to \(T\) | unitless; \(P(T,T)=1\) |
| \(y(0,T)\) | continuously compounded zero rate to \(T\) | per year; \(P(0,T)=e^{-y(0,T)\,T}\) |
| \(y_{sa}(0,T)\) | semiannually compounded zero rate to \(T\) | per year; \(P(0,T)=(1+y_{sa}(0,T)/2)^{-2T}\) |
| \(F(0;T_1,T_2)\) | simply compounded forward rate over \([T_1,T_2]\) | per year; \(1+\tau(T_1,T_2)F=P(0,T_1)/P(0,T_2)\) |
| \(f_c(0;T_1,T_2)\) | continuously compounded forward rate over \([T_1,T_2]\) | per year; \(f_c=(y(0,T_2)T_2-y(0,T_1)T_1)/(T_2-T_1)\) |
| \(\tau(T_1,T_2)\) | year fraction between dates | years; depends on day count |
| \(C_i\) | deterministic cashflow paid at \(T_i\) | currency; positive = receive |
| \(PV_0\) | present value at time 0 | currency |
| \(DV01\) | PV change for rates down 1bp | currency per 1bp; define bump object + sign |

---

## Flashcards

| # | Question | Answer |
|---|----------|--------|
| 1 | What is \(d(T)\) or \(P(0,T)\)? | The present value today of \$1 received at time \(T\). |
| 2 | How do you price deterministic cash flows $\{(T_i, C_i)\}$? | $\sum_i C_i P(0, T_i)$ |
| 3 | What is the law of one price? | Identical cash flows imply identical prices, absent confounding factors. |
| 4 | What are examples of confounding factors? | Liquidity, special financing (repo specials), taxes, credit risk, balance sheet costs. |
| 5 | What is \(P(0,T)\) economically? | The price today of a zero-coupon bond paying \$1 at \(T\). |
| 6 | Under continuous compounding, what is the PV of $A$ at $T$ discounted at rate $R$? | $A e^{-RT}$ |
| 7 | Define the continuously compounded zero rate \(y(0,T)\). | The rate such that \(P(0,T)=e^{-y(0,T)\,T}\). |
| 8 | How do you extract \(y(0,T)\) from \(P(0,T)\)? | \(y(0,T)= -\frac{1}{T}\ln P(0,T)\) |
| 9 | What is the continuous forward rate from zero rates? | \(f_c=\frac{y(0,T_2)T_2-y(0,T_1)T_1}{T_2-T_1}\) |
| 10 | What is the FRA-style forward rate from discount factors? | $F = \frac{1}{\tau}\left(\frac{P(0,T_1)}{P(0,T_2)} - 1\right)$ |
| 11 | What is replication? | Constructing a portfolio with the same cash flows as the target security. |
| 12 | If a claim's market price differs from replication cost, what exists? | An arbitrage opportunity (buy cheap, sell rich). |
| 13 | Why do discount factors fall with maturity (positive rates)? | Money received sooner can earn interest; delayed money is worth less. |
| 14 | Convert 10% semiannual to continuous compounding. | $R_c = 2 \ln(1.05) = 9.76\%$ |
| 15 | When is the spot rate curve upward-sloping? | When forward rates exceed spot rates. |
| 16 | What are Treasury STRIPS? | Zero-coupon securities created by stripping coupon and principal payments from Treasury bonds. |
| 17 | What is the difference between C-STRIPS and P-STRIPS? | C-STRIPS (coupon strips) are fungible across bonds; P-STRIPS (principal strips) are unique to each bond. |
| 18 | How do STRIPS prices relate to discount factors? | $P(0,T) = \text{STRIPS Price}/100$ |
| 19 | Why might STRIPS trade differently from implied zeros? | Liquidity differences, tax treatment (phantom income), reconstitution option. |
| 20 | What does it mean for a bond to trade "cheap"? | Its market price is below its theoretical value (replication cost). |
| 21 | What is OIS discounting? | Using overnight index swap rates to discount collateralized derivatives. |
| 22 | Why did the market shift from LIBOR to OIS discounting? | LIBOR embedded bank credit risk; OIS better approximates the risk-free rate. |
| 23 | What is a CSA? | Credit Support Annex—specifies collateral arrangements for OTC derivatives. |
| 24 | What are limits to arbitrage? | Balance sheet, funding, liquidity, and counterparty constraints that prevent perfect price enforcement. |
| 25 | How can you lock in a forward rate? | Sell units of the longer-dated zero and buy the shorter-dated zero, resulting in a forward loan. |
| 26 | What is this book’s DV01 convention? | \(DV01 := PV(\text{rates down }1\text{bp})-PV(\text{base})\) for a stated bump object. |

---

## Mini Problem Set

### Questions

1. **Discount factor basics.** If $P(0,1) = 0.95$, what is the PV of $200 paid at $T = 1$?

2. **Zero rate extraction.** Using \(P(0,2) = 0.90\), compute the continuously compounded 2-year zero rate \(y(0,2)\).

3. **Forward rate (FRA-style).** If $P(0, 0.5) = 0.98$ and $P(0, 1.0) = 0.95$, compute the simply compounded forward rate $F(0; 0.5, 1.0)$ with $\tau = 0.5$.

4. **Replication.** A claim pays 10 at $T = 1$ and 110 at $T = 2$. Express its price in terms of $P(0,1)$ and $P(0,2)$.

5. **Curve shock intuition.** Given cash flows at $T = 1, 3, 5$ years, which contributes most to PV sensitivity under a parallel rate increase? Why?

6. **Law of one price test.** Two portfolios have identical cash flows but trade at different prices. Construct the arbitrage and state time-0 profit.

7. **Deposit-to-DF conversion.** A 9-month deposit rate is 6% on Actual/360 with 270 days. Using $\tau = 270/360 = 0.75$, compute $P(0, 0.75)$.

8. **Forward from zero rates.** Zero rates (continuous) are \(y(0,1) = 4\%\) and \(y(0,2) = 5\%\). Compute \(f_c(0; 1, 2)\).

9. **Compounding conversion.** Convert 8% quarterly compounding to continuous.

10. **Negative rates.** If the 1-year rate is -0.3% (continuous), what is $P(0,1)$? Interpret economically.

11. **OIS vs. LIBOR impact.** A 10-year swap has PV = $5,000,000 when discounted at LIBOR and PV = $4,950,000 when discounted at OIS. If the swap is collateralized, which valuation is correct? What is the error from using the wrong curve?

12. **STRIPS arbitrage.** A 1-year C-STRIP trades at 96.00 while the implied discount factor from coupon bonds is $d(1) = 0.952$. Is the STRIP rich or cheap? By how much?

13. **Investor choice.** The 6-month spot rate is 5.5% and the 1-year spot rate is 5.2% (both semiannual). What is the implied 6-month forward rate in 6 months? Would you buy the 1-year zero if you expect the 6-month rate to be 5.0% in six months?

14. **Limits to arbitrage.** An off-the-run Treasury trades 5bp cheap to the fitted curve. Explain why this "cheapness" might persist despite the apparent arbitrage opportunity.

15. **(Compute) DV01 of a zero.** A \$100mm zero-coupon cashflow pays at \(T=5\) with discount factor \(P(0,5)=0.8106\). Using the book’s bump object “parallel \(-1\)bp shift in \(y(0,T)\)”, approximate \(DV01\).

### Brief Solutions (1–8, 15)

1. $\text{PV} = 200 \times 0.95 = 190$

2. \(y(0,2) = -\frac{1}{2} \ln(0.90) = -0.5 \times (-0.1054) = 0.0527 = 5.27\%\)

3. $F = \frac{1}{0.5}\left(\frac{0.98}{0.95} - 1\right) = 2 \times (1.0316 - 1) = 6.32\%$

4. $V_0 = 10 \, P(0,1) + 110 \, P(0,2)$

5. The $T = 5$ cash flow contributes most because PV sensitivity is proportional to $T_i \cdot C_i \cdot P(0, T_i)$; the longer maturity (multiplier $T_i$) generally dominates.

6. Buy the cheap portfolio, sell the expensive one. Profit = (expensive price) − (cheap price). Future cash flows net to zero.

7. $P(0, 0.75) = \frac{1}{1 + 0.06 \times 0.75} = \frac{1}{1.045} = 0.9569$

8. \(f_c(0; 1, 2) = \frac{0.05 \times 2 - 0.04 \times 1}{2 - 1} = 0.06 = 6\%\)

15. \(PV_0 = N\cdot P(0,5)=\$100{,}000{,}000\times 0.8106=\$81.06\text{mm}\). First order: \(DV01 \approx PV_0\times T\times 10^{-4}=81.06\text{mm}\times 5\times 10^{-4}\approx \$40.5\text{k}\).

---

## References

- (Bruce Tuckman, *Fixed Income Securities*, “Discount Factors”; “Arbitrage and the Law of One Price”; “Treasury STRIPS”; “Appendix 2A: The Relation between Spot and Forward Rates and the Slope of the Term Structure”; “Yield-Based DV01”)
- (David G. Luenberger, *Investment Science*, “4.6 Running Present Value”)
- (John H. Cochrane, *Asset Pricing*, “Preface > 3.4 Risk Sharing”)
- (Hirsa, *Computational Methods in Finance*, “A.2 Forward Rate Agreement (FRA)”)
- (Hillel Derman, *The Volatility Smile*, “The One Law of Quantitative Finance”)
- (John C. Hull, *Risk Management and Financial Institutions*, “LIBOR vs. Treasury Rates”; “The OIS Rate”)
- (Andersen & Piterbarg, *Interest Rate Modeling*, “6.4.2 Forward Rate Approach”)
